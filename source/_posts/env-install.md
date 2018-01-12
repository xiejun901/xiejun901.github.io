title: 环境安装
date: 2017-11-23 00:23:09
tags: [install, environment, tensorflow, nvidia, cuda]
categories: other
---

# 记录一些软件的安装过程，帮助后续环境的重新安装之类的


## Nvidia 驱动以及cuda，cudnn的安装

看网上说安装驱动的时候可能会出现循环重启，但是看一个人提供了不会出现这种问题的安装方式，就直接使用了，因此并没有感受到循环重启的问题

```shell
wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_8.0.44-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1604_8.0.44-1_amd64.deb
sudo apt-get update
sudo apt-get install cuda
# 环境变量可以写入配置文件中
export PATH=/usr/local/cuda-8.0/bin:PATH
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:LD_LIBRARY_PATH
``` 

这儿有一点需要的是由于由于可能有多个版本，可以通过命令查看版本后安装指定的版本，多个版本安装了之后可以使用软链接进行管理

```shell
apt-cache madison <<package name>>
```
还得再安装cudnn才能使用tensorflow的，可以按照如下的方式安装

```
CUDNN_TAR_FILE="cudnn-8.0-linux-x64-v6.0.tgz"
wget http://developer.download.nvidia.com/compute/redist/cudnn/v6.0/${CUDNN_TAR_FILE}
tar -xzvf ${CUDNN_TAR_FILE}
sudo cp -P cuda/include/cudnn.h /usr/local/cuda-8.0/include
sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda-8.0/lib64/
sudo chmod a+r /usr/local/cuda-8.0/lib64/libcudnn*
```

在找cudnn安装的时候发现一个脚本，集成了以上两步

```shell
#!/bin/bash

# install CUDA Toolkit v8.0
# instructions from https://developer.nvidia.com/cuda-downloads (linux -> x86_64 -> Ubuntu -> 16.04 -> deb (network))
CUDA_REPO_PKG="cuda-repo-ubuntu1604_8.0.61-1_amd64.deb"
wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/${CUDA_REPO_PKG}
sudo dpkg -i ${CUDA_REPO_PKG}
sudo apt-get update
sudo apt-get -y install cuda

# install cuDNN v6.0
CUDNN_TAR_FILE="cudnn-8.0-linux-x64-v6.0.tgz"
wget http://developer.download.nvidia.com/compute/redist/cudnn/v6.0/${CUDNN_TAR_FILE}
tar -xzvf ${CUDNN_TAR_FILE}
sudo cp -P cuda/include/cudnn.h /usr/local/cuda-8.0/include
sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda-8.0/lib64/
sudo chmod a+r /usr/local/cuda-8.0/lib64/libcudnn*

# set environment variables
export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
## anaconda 的安装

官网速度太慢，可以去清华的开源软件镜像站下载，很快。

[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/)

## tensorflow 安装

安装好nvidia驱动之后直接安装官网的方式安装使用pip安装就行

我用的anaconda安装

tfBinaryURL 可以到这儿查看 [URL of the TensorFlow Python package](https://www.tensorflow.org/install/install_linux#the_url_of_the_tensorflow_python_package)

```shell
pip install --ignore-installed --upgrade tfBinaryURL
```

## hexo 的安装

进入官网，查看安装方法

可能需要先安装npm, node.js, node.js 可以使用淘宝的源，会快一些  [node.js淘宝](https://npm.taobao.org/mirrors/node/)

```shell
# 安装node, 版本可以自己选择
wget https://npm.taobao.org/mirrors/node/v9.2.0/node-v9.2.0-linux-x64.tar.gz
tar -xvf node-v9.2.0-linux-x64.tar.gz
sudo mv node-v9.2.0-linux-x64 /opt/
sudo ln -s /opt/node-v9.2.0-linux-x64/bin/node /usr/local/bin/node
sudo ln -s /opt/node-v9.2.0-linux-x64/bin/npm /usr/local/bin/npm
# 如上安装啦npm之后这一步可以不用啦安装 npm 了
sudo apt-get install npm
sudo npm install hexo-cli -g
# 进入到博客的目录
cd PATH
npm install
hexo server
```

然后就启动了，可以使用了。

## hexo mathjax 安装和使用

hexo 使用mathjax需要先更换markdown渲染插件，然后再在设置中设置使用mathjax

```shell
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-pandoc --save
```
然后修改配置文件的一些转意设置
node_modules\kramed\lib\rules\inline.js 这个文件修改如下内容

```
//  escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
  escape: /^\\([`*\[\]()#$+\-.!_>])/,

//  em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
  em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```

打开配置开关

进入主题的目录找到   themes/next/_config.yml 进行修改

```
mathjax:
  enable: true
  per_page: true
```

如果per_page设置成了true, 那么需要在页面开头写下如下语句才能在本页启动mathjax渲染

```
---
title: index.html
date: 2016-12-28 21:01:30
tags:
mathjax: true
--
```

## cisco anyconnect 服务端搭建

[ocserv 安装和配置](https://www.vultr.com/docs/setup-openconnect-vpn-server-for-cisco-anyconnect-on-ubuntu-14-04-x64) 

防丢防打不开，将上面的链接的内容转一份

OpenConnect server, also known as ocserv, is a VPN server that communicates over SSL. By design, its goal is to become a secure, lightweight, and fast VPN server. OpenConnect server uses the OpenConnect SSL VPN protocol. At the time of writing, it also has experimental compatibility with clients that use the AnyConnect SSL VPN protocol.

This article will show you how to install and setup ocserv on Ubuntu 14.04 x64.

Installing ocserv

Since Ubuntu 14.04 does not ship with ocserv, we will have to download the source code and compile it. The latest stable version of ocserv is 0.9.2.

Download ocserv from the official site.

```
wget ftp://ftp.infradead.org/pub/ocserv/ocserv-0.9.2.tar.xz
tar -xf ocserv-0.9.2.tar.xz
cd ocserv-0.9.2
```

Next, install the compile dependencies.

```
apt-get install build-essential pkg-config libgnutls28-dev libwrap0-dev libpam0g-dev libseccomp-dev libreadline-dev libnl-route-3-dev
```

Compile and install ocserv.

```
./configure
make
make install
Configuring ocserv
```

A sample config file is placed under the directory ocser-0.9.2/doc. We will use this file as a template. At first, we have to make our own CA cert and server cert.

```
cd ~
apt-get install gnutls-bin
mkdir certificates
cd certificates
```

We create a CA template file (ca.tmpl) with the content similar to the following. You can set your own "cn" and "organization".

```
cn = "VPN CA" 
organization = "Big Corp" 
serial = 1 
expiration_days = 3650
ca 
signing_key 
cert_signing_key 
crl_signing_key 
```

Then, generate a CA key and CA cert.

```
certtool --generate-privkey --outfile ca-key.pem
certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem
```

Next, create a local server certificate template file (server.tmpl) with the the content below. Please pay attention to the "cn" field, it must match the DNS name or IP address of your server.

```
cn = "you domain name or ip"
organization = "MyCompany" 
expiration_days = 3650 
signing_key 
encryption_key
tls_www_server
```

Then, generate the server key and certificate.

```
certtool --generate-privkey --outfile server-key.pem
certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem
```

Copy the key, certificate, and config file to the ocserv config directory.

```
mkdir /etc/ocserv
cp server-cert.pem server-key.pem /etc/ocserv
cd ~/ocserv-0.9.2/doc
cp sample.config /etc/ocserv/config
cd /etc/ocserv
```

Edit the config file under /etc/ocserv. Uncomment or modify the fields described below.

```
auth = "plain[/etc/ocserv/ocpasswd]"

try-mtu-discovery = true

server-cert = /etc/ocserv/server-cert.pem
server-key = /etc/ocserv/server-key.pem

dns = 8.8.8.8

# comment out all route fields
#route = 10.10.10.0/255.255.255.0
#route = 192.168.0.0/255.255.0.0
#route = fef4:db8:1000:1001::/64
#no-route = 192.168.5.0/255.255.255.0

cisco-client-compat = true
```

Generate a user that will be used to login to ocserv.

```
ocpasswd -c /etc/ocserv/ocpasswd username
```

Enable NAT.

```
iptables -t nat -A POSTROUTING -j MASQUERADE
```

Enable IPv4 forwarding. Edit the file /etc/sysctl.conf.

```
net.ipv4.ip_forward=1
```

Apply this modification.

```
sysctl -p /etc/sysctl.conf
```

Start ocserv and connect using Cisco AnyConnect

First, start ocserv.

```
ocserv -c /etc/ocserv/config
```

Then, install Cisco AnyConnect on any of your devices, such as iPhone, iPad, or an Android device. Since we used a self-signed server key and certificate, we have to uncheck the option which prevents insecure servers. This option is located in the settings of AnyConnect. At this point, we can setup a new connection with the domain name or IP address of our ocserv and the username/password that we created.

Connect and enjoy!


