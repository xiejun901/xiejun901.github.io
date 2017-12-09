title: 强化学习-DP, MC, TD
date: 2017-08-27 20:11:23
tags: [强化学习, 机器学习, DP, MC, TD]
categories: machine learning
---

上一份笔记记录了强化学习的一些基本概念，其中提到了最优策略，本次笔记记录的就是求解在指定策略下的状态值函数(动作值函数)和最优策略的方法。 

在众多求解最优策略的方法中，求解过程都可以划分成两个部分，策略求值(policy evaluation)和策略提升(policy improvement). 其中策略求值的方法有很多种，主要包括DP，MC, TD 等，策略提升一般使用贪心，$\epsilon-greedy$.

![](http://7q5fny.com1.z0.glb.clouddn.com/rl1.png)

![](http://7q5fny.com1.z0.glb.clouddn.com/rl2.png)

如上图，途途中的$q_\pi$同事可以替换为$v_\pi$，策略求值可以计算到再指定策略下状态值函数(动作值函数)收敛也可以是仅仅迭代一次或者多次之后就进行策略提升。

### DP

####  prediction

动态规划的求值利用的bellman公式进行计算, 是一个迭代过程。通过反复的迭代，可以收敛至$v_\pi$

$$
v_{k+1}(s)=\sum_{a \in A}{\pi(a|s)(R_s^a + \gamma \sum_{s \in S^\prime}{P_{ss^\prime} v_k(s^\prime)})}
$$

#### control

可以反复迭代evaluation过程收敛至$v_\pi$之后进行policy improvement. 新的策略更新为 对于指定的状态s, 选择后继$v(s^\prime)$最大的状态的action, 这种做法成为策略迭代，在迭代的过程中有显示的策略存在

同时, 也可以进行一次计算就更新策略,这种做法没有显示的策略存在，称为值迭代。计算公式如下:

$$
v_{k+1}(s)=\mathop {max}_{a \in A}(R_s^a+\gamma \sum_{s^\prime \in s}{P_{ss^\prime}^a{v_k(s^\prime)}})
$$

### MC

#### prediction

上面的动态规划里面很核心的东西是需要知道状态之间的转移概率，这个在很多实际情况下都是完全不能的，因此希望能通过采样来进行需要，能够从经验中来进行学习。

对于基于 $\pi$ 产生的很多个episodes, 我们可以用多个episodes的reture的均值来作为状态的$v$

S1,A1,R1,S2,A2,R2,S3,A3,R3 ... 

$$
v_\pi(s)=E[G_t|S_t=s]
$$

实际上，我们可以使用如下公式进行计算

$$
v(S_t) = v(S_t) + \alpha (G_t - v(S_t))
$$

同时，也可以使用q函数来进行计算

$$
q(s,a) = q(s,a) + \alpha (G_t - q(s, a))
$$

#### control

同样可以使用计策略求值，策略提升的方式来求最佳策略，但是我们需要做 model free 的最优策略计算，可以看看下面公式

$$
\pi_\*(s)=\mathop{argmax}_{a \in A}(R_s^a + \sum_{s' \in S}{P_{ss^\prime}^a V(s^\prime)})
$$

$$
\pi_\*(s)=\mathop {argmax}_{a \in A}Q(s,a)
$$

显然，没办法使用 V 来做策略提升， 因为使用V来做策略提升就必须知道下一步应该会走到哪些$s^\prime$,这是模型的一部分。
因此只能使用 Q 来进行 control

使用 monte-carlo 来做control事下面这样的过程

- 使用策略 $\pi$ 产出一个episode, S1,A1,R1,S2,A2,R2........
- 针对每个 t 计算 出 $G_t$ 
- 针对每个状态，统计出计算次数和 return 的期望

$$
N(S_t, A_t) = N(S_t, A_t) + 1
$$

$$
Q(S_t, A_t) = Q(S_t, A_t) + \frac{1}{N(S_t, A_t)}(G_t - Q(S_t, A_t))
$$

- 根据当前的Q值使用 $\epsilon-greedy$ 改善策略 

### TD

#### prediction

MC 会收敛得比较慢，因此有了Temporal-Difference (TD), TD的迭代更新方式如下：

$$
V(S_t) = V(S_t) + \alpha (R_{t+1} + \gamma V(S_{t+1}) - V(S_t))
$$



其中$R_{t+1} + \gamma V(S_{t+1})$ 成为 TD target , $R_{t+1} + \gamma V(S_{t+1}) - V(S_t)$ 称为 TD error

当然也可以是用Q函数进行更新

$$
Q(S_t, A_t) = Q(S_t, A_t) + \alpha (R_{t+1} + \gamma Q(S_{t+1}, A_{t+1} - Q(S_{t}, A_{t}))
$$

#### SARSA

同样可以使用策略求值和策略提升来得到最佳策略。由于使用V值计算无法做到model free， 因此直接使用Q值进行距离，计算过程如下:

1 对于初始状态$s$, 使用 $\epsilon-greedy$ 选定动作 $a$, 对应的 $Q$ 值为 $Q(s, a)$. 
2 执行动作$a$, 达到状态$s^\prime$, 得到回报$R$. 根据当前的动作值函数, 使用 $\epsilon-greedy$ 选出动作 $a^\prime$, 通过$\epsilon-greedy$, 达到了策略提升的效果
3 使用如下公式进行迭代, 这个过程便是策略求值

$$
Q(s, a) = Q(s, a) + \alpha (R + \gamma Q(s^\prime, a^\prime) - Q(s, a))
$$

4 将 $s^\prime$, $a^\prime$, 作为新的起点, 替代调$s$ $a$, 执行第二步, 一直迭代到收敛为止, 得到 $Q_{\*}$, 如果在这一步进入到终止状态，那么只需要回到第一步重新选取 $s$ 和 $a$ 就行。

这种算法称为Sarsa, 这种更新方式是 On-Policy 的, 产生动作 $a$ 和 更新 $Q$ 值用到的 $Q(S^\prime, a^\prime)$ 中的 $a^\prime$ 是同一个策略产生的(基于相同的Q函数, 因此是同一个策略), 即产出动作的策略和更新时使用的策略是同一个。

#### On-policy 和  Off-policy

上面提到的方法产生行为的策略和求值的策略是相同的，这种称为on-policy, 但基于以下几个原因，我们有时候需要通过$\mu(a|s)$产生的行为来对目标策略$\pi(a|a)$求值进而求出$v_\pi(s)$ 和 $q_\pi(s,a)$, 这种做法称为 Off-Policy.

1. 可以从其他agent的行为来进行学习，比如可以根据人的行为来让机器人学习策略
2. 可以复用经验数据。
3. 可以通过探索性的策略来得到最优策略。
4. 可以通过一个策略来学习到多个策略。

上文提到的SARSA 就是 On-policy 的一个例子。

不同分布的期望:

$$
\begin{align}
E_{X∼P}[f(X)] &= \sum P(X)f(X) \\
&= \sum Q(X)\frac{P(X)}{Q(X)}f(X) \\
&= \sum \frac{P(X)}{Q(X)}Q(X)f(X) \\
&= E_{X∼Q}[\frac{P(X)}{Q(X)}f(X)]
\end{align}
$$

也就是说根据这个公式，我们可以通过按照 Q 分布产生出的 return 的期望, 来计算得到如果是按照P分布应该得到的 return 的期望。

对于 Monte-Carlo 方法, 经过重要性采样的的 return 如下:

$$
G_t^{\pi/\mu}=\frac{\pi(A_t|S_t) \pi(A_{t+1}|S_{t+1}) ... \pi(A_T|S_t)}{\mu(A_t|S_t) \mu(A_{t+1}|S_{t+1}) ... \mu(A_T|S_t)} G_t
$$

然后进行迭代更新:

$$
V(S_t)=V(S_t) + \alpha (G_t^{\pi/\mu} - V(S_t))
$$

但是随着多步的运算，会带来很高的方差，因此这个方法是没用的。

做Off-Policy需要使用 TD-Learning, TD-Learning 只需要进行一步的重要性采样。

$$
V(S_t) = V(S_t) + \alpha(\frac{\pi(A_t|S_t)}{\mu(A_t|S_t)}(R + \gamma V(S_{t+1})) - V(S_t))
$$

当然，更好的使用Off-Policy的方式是Q-learning

#### Q-Learning

Q-Learning 的几个特点

1 通过Off-Policy来学习Q(s, a)
2 不需要重要性采样
3 行为通过$A_{t+1} ∼ \mu( \cdot |S_t) $ 来产生
4 通过我们需要求值的策略 \pi 来产生一个后继 action $A^\prime$, 注意这个action并不需要真正执行。
5 使用 $A^\prime$ 来更新Q(S_t, A_t)的值。

总结一下就是使用 $\mu$ 来产生行为($A_t$)并执行动作。使用 $\pi$ 来对策略进行求值。这是一个典型的 Off-Policy

更新Q(S_t, A_t) 的公式如下:

$$
Q(S_t, A_t) = Q(S_t, A_t) + \alpha (R_{t+1} + \gamma Q(S_{t+1}, A^\prime) - Q(S_T, A_t))
$$

在进行 control 的时候，需要对策略进行提升，在Q-Learning中, 对 $\pi$ 和 $\mu$ 都进行提升, $\mu$ 使用 $\epsilon-greedy$, 可以保证可以访问到所有的状态, $\pi$使用 greedy 来保证策略的提升。

$$
A^\prime=\mathop{argmax}_{a^\prime \in A} Q(S_{t+1}, a^\prime)
$$

显然

$$
Q(S_{t+1}, A^\prime) = maxQ(S_{t+1}, a^\prime)
$$

因此更新公式为:

$$
Q(S_t, A_t) = Q(S_t, A_t) + \alpha (R_{t+1} + \gamma maxQ(S_{t+1}, a^\prime) - Q(S_T, A_t))
$$

结合前面的内容，Q-Learning 可以描述为:

1 对于状态$S$, 使用 $\epsilon-greedy$ 根据$Q(S, A)$ 选出动作 $A$, 执行动作A, 状态转移为$S^\prime$, 观察到的reward 是 R --- 这一步使用的策略是产生行为的策略$\mu$
2 使用更新公式更新$Q(S, A)$    ---- 这一步使用的是策略 $\pi$ 来进行更新
3 将$S^\prime$ 作为 $S$, 执行步骤1, 如果这个地方$S$是结束状态，需要重新选定一个状态作为$S$ 进行 步骤1, 直到收敛为止。


#### $TD(\lambda)$

在之前的TD-Learning中，使用的TD target 是 $R_{t+1} + \gamma V(S_{t+1})$, 是使用了一步之后的$V$, 这儿也是可以使用多步之后的$V$的, 当取无穷多步的时候，就等效于MC了。

$$
G_t^{(n)}=R_{t+1} + \gamma R_{t+2} + \gamma ^2 R_{t+3} + ... + \gamma ^ {n-1} R_{t+n} + \gamma ^n V(S_{t+n}
$$

$G_t^{(n)}$ 成为n-step return, n取1的时候就是TD, 取无穷的时候就是MC

$$
V(S_t) = V(S_t) + \alpha(G_t^{(n)} - V(S_t))
$$

可以将多个 n-step return 进行叠加作为 return来进行计算。定义如下加权平均

$$
G_t^\lambda = (1-\lambda)\sum_{n=1}^{n=\infty}{\lambda^{n-1} G_t^{(n)} }
$$

可以使用 $G_t^\lambda$ 来进行 V 值的更新

$$
V(S_t) = V(S_t) + \alpha(G_t^\lambda - V(S_t))
$$

这种直接更新的方式成为 forward-view $TD(\lambda)$, 但是实际中这个不好用，因为要用到未来的数据，也就是说如果episode是无限的或者想要做online的更新，这就没法实现。因此实际中会使用 backward-view $TD(\lambda)$ , 两种方式的计算原理是一样的，只是改变了计算方式。

对于 backward-view $TD(\lambda)$ , 记录以下项

$$
E_0(s) = 0
$$

$$
E_t(s) =\gamma \lambda E_{t-1}(s) + 1(S_t=s)
$$

$$
\delta_t=R_{t+1} + \gamma V(S_{t+1}) - V(S_t)
$$

$$
V(S_t) = V(S_t) + \alpha \delta_t E_t(S_t)
$$

注意对于 backward-view $TD(\lambda)$, 访问一个状态的时候, 需要对所有状态的$E(s)$和$V(s)$ 进行更新。

当然, 上面公式中的$V$也是可以使用$Q$的

$$
Q_0(s, a) = 0
$$

$$
Q_t(s, a) = \gamma \lambda Q_{t-1}(s, a) + 1(S_t=s, A_t=a)
$$

$$
\delta_t=R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t)
$$

$$
Q(S_t, A_t) = Q(S_t, A_t) + \alpha \delta E_t(S_t, A_t)
$$

将这个公式套入SARSA中就得到了SARSA($\lambda$). SARSA($\lambda$) 如下:

1 对每个状态初始化$E(s, a)$ , $Q(s, a)$
2 随机选择一个状态$s$作为初始状态, 然后使用$\epsilon-greedy$选出动作$a$。
3 在状态$s$上执行动作$a$, 状态会转移到$s^\prime$, 得到回报 $R$, 使用$\epsilon-greedy$ 选取动作$a^\prime$.
4 更新 

$$
E(s, a)= \gamma \lambda E(s, a) +1
$$

$$
\delta = R + \gamma Q(s^\prime, a^\prime) - Q(s, a)
$$

对于所有状态进行更新:
$$
Q(S, A) = Q(S, A) + \alpha \delta E(S, A)
$$

$$
E(S, A)= \gamma \lambda E(S, A)
$$

5 $s=s^\prime$ , $a=a^\prime$, 然后迭代执行2










