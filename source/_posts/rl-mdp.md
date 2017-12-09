title: 强化学习-MDP
date: 2017-08-21 23:10:23
tags: [强化学习, 机器学习, MDP]
categories: machine learning
---

### markov process

马尔科夫过程是一类随机过程，对于已知目前的状态下，其下一状态与过去的状态无关，仅仅与现在的状态有关。即:

$$
P(S_{t+1}|S_{t})=P(S_{t+1}|S_t, S_{t-1}, S_{t-2} ..., S_t)
$$

对于一个马尔科夫过程，可以使用状态转移概率和状态转移矩阵来描述

$$
p_{ss^\prime}=P(S_{t+1}|S_{t})
$$

$$
P = \begin{bmatrix} p_{11}, p_{12}, ..., p_{1n} \\
p_{21}, p_{22}, ..., p_{2n} \\ p_{m1}, p_{m2}, ..., p_{mn} \end{bmatrix}
$$

### markov reward process

在马尔科夫过程中加入 reward 就构成了 MRP, 状态从S转移到$S^\prime$ , 出了状态变更外，还会获得一个奖励 Reward.

#### 几个基本的定义

$R_t$ : 从状态 $S_t$ 到 $S_{t+1}$ 获得的奖励
$R_s=E(R_t+1|S_t=s)$
$G_t=R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + ... =\sum_{k=0}^{\infty}{\gamma^k R_{t+k+1}}
$
$v(s)=E(G_t|S_t=s)$ : 值函数, 表示在状态 s 未来获得 reward 的期望值。

#### bellman 方程

$$
\begin{align}
v(s) &= E(G_t|s_t=s) \\
&= E[R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + ... |s_t=s ] \\
&= E[R_{t+1} + \gamma (R_{t+2} + \gamma R_{t+3} + \gamma^2 R_{t+4} ...)|s_t=s] \\
&=E[R_{t+1} + \gamma (V(s_{t+1})|s_t=s]
\end{align}
$$

考虑状态转移概率

$$
v(s) = R(s) + \gamma \sum_{s^\prime \in S}{P_{ss^\prime}v(s^\prime)}
$$


                   v(s)  o
                       /   \
               v(s‘)  o     o

需要注意一下的是，上面的式子中，$R(s)$是定义到S上面的，是已经求过期望的，表达的是在状态s上获得的奖励。

### markov decision process

#### 几个基本的定义

MDP 是在 MRP 的基础上增加了决策，可以用一个<S,A,P,R,$\gamma$>来表示

其中S表示状态，A表示action, P是一个状态转移矩阵，表示在指定action下，从状态$s$转移到$s^\prime$的概率,$R$表示在指定状态执行指定action可以获得的奖赏。

$$
P_{ss^\prime}^a=P[S_{t+1}=s^\prime|S_t=s,A_t=a]
$$

$$
R_s^a=E[R_{t+1}|S_t=s,A_t=a]
$$

策略$\pi$表示的是一个从$s$到$a$的映射，对于每个状态，应该怎么选择action

$$
\pi(a|s)=P(A_t=a|S_t=s)
$$

在策略$\pi$下，状态s的值函数 state-value

$$
v_\pi(s)=E[G_t|S_t=s]
$$

动作q的值函数 action-value

$$
q_\pi(s,a)=E[G_t|S_t=s,A_t=a]
$$

#### bellman 方程 

对于 MDP, 同样存在bellman 方程

$$
v_\pi(s)=E_\pi[R_{t+1} + \gamma v_\pi(S_{t+1})|s=S_t]
$$

$$
q_\pi(s,a)=E_\pi[R_{t+1} + \gamma q_\pi(S_{t+1},A_{t+1})|S_t=s,A_t=a]
$$

            v(s) <- s    o
                       /   \
          q(s,a) <- a .     .

$$
v_\pi(s)=\sum_{a \in A}{\pi(a|s) q(s,a)}
$$

           q(s,a) <- a   .
                       /   \
        v(s') <- s'   o     o

$$
q(s,a)=R_s^a + \gamma \sum_{s^\prime \in S}P_{ss\prime}^a{v_\pi(s^\prime)}
$$

            v(s) <- s    o
                       /   \
                      .     .
                  r  / \   / \
       v(s') <- s'  o   o o   o


$$
v_\pi(s)=\sum_{a \in A}{\pi(a|s)}(R_s^s + \gamma \sum_{s^\prime \in S}{P_{ss^\prime}^a v_\pi(s^\prime)})
$$

              q(s,a) <- a    .
                      r    /   \
                          o     o
                         / \   / \
        q(s',a') <- a'  .   . .   .

$$
q_\pi(s,a)=R_s^a + \gamma \sum_{s^\prime \in S}{P_{ss^\prime}^a \sum_{a^\prime \in A}{\pi(a^\prime|s^\prime)q_\pi(s^\prime,a^\prime)}}
$$


### 最优策略

最优状态值函数

$$
v_*(s)=max_{\pi} v_\pi(s)
$$

最优动作值函数

$$
q_*(s,a)=max_\pi {q_\pi(s,a)}
$$

最优策略，最优动作值函数确定后，最优策略就确定了, 对于状态s, 只需要选择q最大的那个动作就行了。


#### 最优策略的bellman 方程

                v*(s) <- s    o
                            /   \
              q*(s,a) <- a .     .


$$
v_\*(s)=\mathop {max}_{a} q_{\*}(s,a)
$$

           q*(s,a) <- a   .
                        /   \
        v*(s') <- s'   o     o

$$
q_\*(s,a)=R_s^a + \gamma \sum_{s^\prime \in S}{P_{ss^\prime}^a v_\*(s^\prime)}
$$


            v*(s) <- s    o
                        /   \
                       .     .
                   r  / \   / \
       v*(s') <- s'  o   o o   o

$$
v_\*(s)=\mathop {max}_{a} [R_s^a + \gamma \sum_{s^\prime \in S}{P_{ss^\prime}^a v_\*(s^\prime)}]
$$

              q*(s,a) <- a    .
                       r    /   \
                           o     o
                          / \   / \
        q*(s',a') <- a'  .   . .   .

$$
q_\*(s,a)=R_s^a + \gamma \sum_{s^\prime \in S}{P_{ss^\prime}^a \mathop {max}_{a^\prime}{q_\*(s^\prime,a^\prime)}}
$$
