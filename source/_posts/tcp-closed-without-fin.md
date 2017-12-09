title: Linux 下 TCP 连接断开未发送 FIN
date: 2016-01-12 00:36:51
tags: [TCP, socket, Linux, FIN, RST]
categories: UNP
---

最近写一个简单的聊天服务器时候， 看了UNP上通过带外数据想实现心跳的例子，就尝试了使用一下，然后之前好好的客户端崩溃掉服务端也能正常处理，加了这块代码之后客户端崩溃服务端也会得到 Connection reset by peer 的错误。 
经过各种纠结， 最后在群友的帮助下，发现了是由于在Linux下，如果TCP接收缓冲区中由数据，那么此时调用close，便不会发送Fin， 会直接发送RST。由于通过select监听带外数据异常，在通知带外数据异常时，虽然调用recv(1, MSG_OOB)读走了带外数据，但是由于没有调用普通的读数据，还是会被当做缓冲区由数据，直接发送RST而不发送FIN
在这儿记录以下找到问题的过程


简化了的出问题的代码

```python
# 客户端代码
import socket
import time
import select

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.bind(('localhost', 8081))
sock.connect(('localhost', 8080))
connections = {}
connections[sock.fileno()] = sock
expires = time.time()
while True:
    r = []
    e = []
    for fileno, conn in connections.items():
        r.append(fileno)
        e.append(fileno)
    r, w, e = select.select(r, [], e, 1)
    for fileno in r:
        conn = connections[fileno]
        print(conn.recv(1024))
    for fileno in e:
        conn = connections[fileno]
        print(conn.recv(1, socket.MSG_OOB))
    current = time.time()
    while current> expires:
        expires += 1000
        sock.send('c', socket.MSG_OOB)
		
# 服务端代码
import socket
import select

try:
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.setblocking(0)
    sock.bind(('localhost', 8080))
    sock.listen(5)
    connections = {}
    connections[sock.fileno()] = sock
    while True:
        r = []
        w = []
        e = []
        for fileno, conn in connections.items():
            r.append(fileno)
            e.append(fileno)
        r, w, e = select.select(r, w, e)
        for fileno in r:
            conn = connections[fileno]
            if conn == sock:
                conn, addr = sock.accept()
                connections[conn.fileno()] = conn
            else:
                data = conn.recv(1024)
                if data is None:
                    conn.close()
                    del connections[fileno]
        for fileno in e:
            conn = connections[fileno]
            print(conn.recv(1, socket.MSG_OOB))
            conn.send('c', socket.MSG_OOB)
finally:
    sock.close()
```

遇到问题之后，想了各种可能的情况，比如对方已经发送FIN了，还朝对方发送了数据或者接收了数据之类的，最后想到好像有个tcpdump命令可以看tcp的通信情况，就使用这个命令看了以下tcp双方收发的数据情况，发现client只发送了RST没发送FIN，这和之前印象中看到的结束进程会发送FIN不符啊，翻了翻UNP、确实没找到相关内容，就去某群里问了一下什么时候会出现不发送FIN

	03:04:09.137374 IP localhost.tproxy > localhost.http-alt: Flags [S], seq 2126650160, win 43690, options [mss 65495,sackOK,TS val 113521554 ecr 0,nop,wscale 7], length 0
	03:04:09.137415 IP localhost.http-alt > localhost.tproxy: Flags [S.], seq 2461040723, ack 2126650161, win 43690, options [mss 65495,sackOK,TS val 113521555 ecr 113521554,nop,wscale 7], length 0
	03:04:09.137446 IP localhost.tproxy > localhost.http-alt: Flags [.], ack 1, win 342, options [nop,nop,TS val 113521555 ecr 113521555], length 0
	03:04:10.137866 IP localhost.tproxy > localhost.http-alt: Flags [P.U], seq 1:2, ack 1, win 342, urg 1, options [nop,nop,TS val 113521805 ecr 113521555], length 1
	03:04:10.137976 IP localhost.http-alt > localhost.tproxy: Flags [.], ack 2, win 342, options [nop,nop,TS val 113521805 ecr 113521805], length 0
	03:04:10.138218 IP localhost.http-alt > localhost.tproxy: Flags [P.U], seq 1:2, ack 2, win 342, urg 1, options [nop,nop,TS val 113521805 ecr 113521805], length 1
	03:04:10.138258 IP localhost.tproxy > localhost.http-alt: Flags [.], ack 2, win 342, options [nop,nop,TS val 113521805 ecr 113521805], length 0
	03:04:37.620543 IP localhost.tproxy > localhost.http-alt: Flags [R.], seq 2, ack 2, win 342, options [nop,nop,TS val 113528675 ecr 113521805], length 0
	
从上面的tcpdump信息可以发现，客户端发起连接后，向服务端发送了SYN， 服务端回 SYN+ACK， 客户端回ACK， 客户端发送带外数据，服务端收到带外数据发送ACK， 服务端发送带外数据，客户端收到带外数据回复ACK， 然后此时我kill掉了客户端进程，此时并没有发送FIN， 直接发送了RST，此时服务端的select会检测到可读事件，调用recv读取一个已经接收到RST的socket就会报ECONNRESET错误

后面有人提出了说在Linux下，如果TCP缓冲区里还由内容没吸干净，调用close就会出现发送RST而不发送FIN的情况，还给出了代码复现，找到了源代码中相关内容，然后其他人还找出了相关文档 [TCP RST: Calling close() on a socket with data in the receive queue](http://cs.ecs.baylor.edu/~donahoo/practical/CSockets/TCPRST.pdf)


如下代码, 使用tcpdump 也出现了同样的问题

```C
//client
int main() {
    char buff[] = "hello";
    int clientfd = socket(AF_INET, SOCK_STREAM, 0);
    sockaddr_in serveraddr;
    bzero(&serveraddr, sizeof(serveraddr));
    serveraddr.sin_family = AF_INET;
    serveraddr.sin_port = htons(8080);
    inet_pton(AF_INET, "127.0.0.1", &serveraddr.sin_addr);
    connect(clientfd, (struct sockaddr* )&serveraddr, sizeof(serveraddr));
    sleep(3);
    //send(clientfd, buff, strlen(buff), 0);
    close(clientfd);
    return 0;
}

//server 
int main() {
    char buff[10]="hello";
    bzero(buff, sizeof(buff));
    int listenfd = socket(AF_INET, SOCK_STREAM, 0);
    sockaddr_in serveraddr;
    bzero(&serveraddr, sizeof(serveraddr));
    serveraddr.sin_family = AF_INET;
    serveraddr.sin_port = htons(8080);
    inet_pton(AF_INET, "127.0.0.1", &serveraddr.sin_addr);
    bind(listenfd, (struct sockaddr *)&serveraddr, sizeof(serveraddr));
    listen(listenfd, 5);
    sockaddr_in cliaddr;
    socklen_t len;
    int connfd = accept(listenfd, (struct sockaddr *)&cliaddr, &len);
    //int n = recv(connfd, buff, 10, 0);
    send(connfd, buff, strlen(buff), 0);
    //buff[n] = 0;
    cout<<buff<<endl;
    close(listenfd);
    close(connfd);
    return 0;
}
```

如果修改客户端代码读取掉数据就能正常的发送FIN， 

在 /net/ipv4/tcp.c 中2063行左右有相关逻辑， 估摸着发送了带外数据，调用recv(fd, 1, MSG_OOB) 并不能让 data_was_unread 为 0， 具体细节可能得更后面再去了解了

```c
	while ((skb = __skb_dequeue(&sk->sk_receive_queue)) != NULL) {
		u32 len = TCP_SKB_CB(skb)->end_seq - TCP_SKB_CB(skb)->seq;

		if (TCP_SKB_CB(skb)->tcp_flags & TCPHDR_FIN)
			len--;
		data_was_unread += len;
		__kfree_skb(skb);
	}

	sk_mem_reclaim(sk);

	/* If socket has been already reset (e.g. in tcp_reset()) - kill it. */
	if (sk->sk_state == TCP_CLOSE)
		goto adjudge_to_death;

	/* As outlined in RFC 2525, section 2.17, we send a RST here because
	 * data was lost. To witness the awful effects of the old behavior of
	 * always doing a FIN, run an older 2.1.x kernel or 2.0.x, start a bulk
	 * GET in an FTP client, suspend the process, wait for the client to
	 * advertise a zero window, then kill -9 the FTP client, wheee...
	 * Note: timeout is always zero in such a case.
	 */
	if (unlikely(tcp_sk(sk)->repair)) {
		sk->sk_prot->disconnect(sk, 0);
	} else if (data_was_unread) {
		/* Unread data was tossed, zap the connection. */
		NET_INC_STATS_USER(sock_net(sk), LINUX_MIB_TCPABORTONCLOSE);
		tcp_set_state(sk, TCP_CLOSE);
		tcp_send_active_reset(sk, sk->sk_allocation);
	} else if (sock_flag(sk, SOCK_LINGER) && !sk->sk_lingertime) {
		/* Check zero linger _after_ checking for unread data. */
		sk->sk_prot->disconnect(sk, 0);
		NET_INC_STATS_USER(sock_net(sk), LINUX_MIB_TCPABORTONDATA);
	} else if (tcp_close_state(sk)) {
```