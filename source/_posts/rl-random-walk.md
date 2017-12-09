title: 强化学习-Random Walk 实例
date: 2017-09-11 00:24:16
tags: [强化学习, 机器学习, Random Walk, DP, SARSA, TD, MC]
categories: machine learning
---

前面的几个笔记讲了强化学习的一些基本概念和算法, 上一篇的符号都有些混乱了, 这一篇以 random walk 举例, 再进一步针对具体的问题描述各个算法, 起到一个回顾和加深理解的作用。
为什么要选择 random walk 呢, 因为这个问题足够简单, 更有利于理解基本概念和问题的本质。

### 基本概念

![](http://7q5fny.com1.z0.glb.clouddn.com/randomwalk.png)

如上图所示，一共存在A, B, C, D, E 五个状态和两个终止状态。对于五个状态存在向左和向右两种动作。到达非终止状态的时候获得得回报都是0, 只有到达最右边的终止状态的时候能得到1的回报。

一些基本概念:

**State**: A, B, C, D, E都是状态。 $s \in {\\{ A, B, C, D, E\\} }$
**Action**: 一共有两个动作, 向左，向右。 $a \in { \\{向左, 向右 \\}}$
**episode**: 从MDP中产出的一个序列, 例如:

```
C->D->E->END2
C->B->A->B->A->B->A->END1
C->B->C->D->C->B->A->A->B->C->D->E->END2
```

**策略**: 从状态产生动作的函数, 例如随机策略, 在每个状态上都是以0.5的概率选作向左和向右

$\pi(A=向左|S=E)=0.5$ ,  $\pi(A=向右|S=E)=0.5$

**Return**: 对于一个指定的episode, 可以计算得到一个return

对于
```
C->D->E->END2
```

$$G=0+\gamma \* 0 + \gamma^2 \* 1$$

**State-value function** : 对于指定的策略，可以求得每个状态上将来能获得得回报

$$
V_\pi(s)=E_\pi[G_t|S_t=s]
$$

**Action-value function** : 同样, 对于指定的策略，可以求得每组动作-状态未来可以获得得回报

$$
Q_\pi(s, a) = E_\pi[G_t|S_t=s, A_t=a]
$$

在后文中, 需要示例对指定状态求值的时候, 都是通过随机策略进行举例。

**最优策略** : 在所有的策略中，能够使得所有的状态都取得最大的状态值函数或者动作值函数的的策略被称为最优策略。对应的值函数称为最优状态值函数和最优动作值函数。整个强化学习的核心内容就是求解最优策略。

### 算法

#### 直接求解

对于已知的, 简单的MDP, 实际上是可以直接求得指定策略下的状态值函数。对于random walk, 可以通过bellman 方程列出如下式子

状态值函数的 bellman 方程

$$
V(s)=\sum_{a \in A}(R_s^a + \gamma \sum_{s^\prime \in S}P_{ss^\prime}^a V(s^\prime))
$$

对于每个状态都可以使用 bellman 方程列出式子, 然后联立求解。

$$
\left\\{ 
\begin{array}{cc}
V(A) = 0.5(0 + \gamma \ast 0 ) + 0.5(0 + \gamma \ast V(B)) \\
V(B) = 0.5(0 + \gamma \ast V(A)) + 0.5(0 + \gamma \ast V(C)) \\
V(C) = 0.5(0 + \gamma \ast V(B)) + 0.5(0 + \gamma \ast V(D)) \\
V(D) = 0.5(0 + \gamma \ast V(C)) + 0.5(0 + \gamma \ast V(E)) \\
V(E) = 0.5(0 + \gamma \ast V(D)) + 0.5(1 + \gamma \ast 0)
\end{array}
\right.
$$

求解以上式子就可以得到:

$$
\left\\{ 
\begin{array}{c}
V(A) = \frac{1}{6} \\ 
V(B) = \frac{1}{3} \\ 
V(C) = \frac{1}{2} \\
V(D) = \frac{2}{3} \\
V(E) = \frac{5}{6}
\end{array}
\right. 
$$

同样也可以使用动作值函数的bellman方程列出等式, 然后求解, 不过由于需要一组(s, a)才能对应到Q，因此列出的等式更多, 求解的复杂度更高。

$$
Q(s, a) = R_s^a + \gamma \sum_{s^\prime \in S} P_{ss^\prime}^a \sum_{a^\prime \in A} \pi(a^\prime | s^\prime) Q(s^\prime, a^\prime)
$$

#### DP

直接计算的具有较高的复杂度。可以利用 bellman 方程迭代求解。首先介绍一下使用DP进行策略评估的算法。

这儿试用计算$v_\pi$举例
更新公式如下:

$$
v(s) = \sum_{a \in A}{\pi(a|s)(R_s^a + \gamma \sum_{s^\prime \in S}P_{ss^\prime}^a v(s^\prime))}
$$

```python
v=[0, 0, 0, 0, 0, 0, 0]
gamma=1
for i in range(500):
    for j in range(1, 6):
        if(j == 5):
            v[j] = 0.5 * (0 + gamma * v[j-1]) + 0.5 * (1 + gamma * 0)
        else:
            v[j] = 0.5 * (0 + gamma * v[j-1]) + 0.5 * (0 + gamma * v[j+1])
print(v)
```

可以看到结果为

```
[0, 0.16666666666666663, 0.33333333333333326, 0.4999999999999999, 0.6666666666666665, 0.8333333333333333, 0]
```

与上一部直接计算的基本一致

#### MC-Prediciton

上面讲到的DP有很多的缺陷，比如必须要知道各个状态间的转移概率，而 MC
使用采样来进行求解。通过与环境交互或者一个个episode, 然后使用各个 episode 的 return 的均值作为状态的值函数。

使用如下的更新公式:

$$
v(s) = v(s) + \alpha(G_t - v(s))
$$

```python
v=[0.0, 0.5, 0.5, 0.5, 0.5, 0.5, 1.0]
gamma=1
alpha = 0.0001
def get_episode():
    s = 3
    res = []
    reward = []
    res.append(s)
    while True:
        action = random.randint(0,2)
        if(action == 0):
            s -= 1
        else:
            s += 1
        res.append(s)
        if(s == 6):
            reward.append(1.0)
            break
        elif(s == 0):
            reward.append(0.0)
            break
        reward.append(0.0)
    return res, reward
for n in range(100000):
    states, rewards = get_episode()
    g=0.0
    for i in range(len(states)-1-1, 0, -1):
        g = gamma * g + rewards[i]
        v[states[i]] = v[states[i]] + alpha * (g - v[states[i]])
print(v)
```

#### MC-Control

强化学习的最终目的是要学得最优策略，因此在进行策略评估之后还需要对策略进行改进, 策略改进可以是在策略求值到收敛之后进行, 也可以计算一步就更新。

对于 MC-Control 与 MC-Prediction 不一样的仅仅是产生 episode 的时候要用更新后的策略，不再是一直使用指定策略。

这个地方将$gamma$设置为小于1才能体现出各个状态之间的差别。

```python
v=[0.0, 0.5, 0.5, 0.5, 0.5, 0.5, 1.0]
gamma = 0.9
alpha = 0.0001
epsilon = 0.05
def get_episode():
    s = 3
    res = []
    reward = []
    res.append(s)
    while True:
        action = 1
        if(v[s-1] > v[s+1]):
            action = 0
        if(random.random() < epsilon):
            action = random.randint(0,2)
        if(action == 0):
            s -= 1
        else:
            s += 1
        res.append(s)
        if(s == 6):
            reward.append(1.0)
            break
        elif(s == 0):
            reward.append(0.0)
            break
        reward.append(0.0)
    return res, reward
for n in range(10000000):
    states, rewards = get_episode()
    g=0.0
    for i in range(len(states)-1-1, 0, -1):
        g = gamma * g + rewards[i]
        v[states[i]] = v[states[i]] + alpha * (g - v[states[i]])
print(v)
```

#### TD-Prediction

MC 需要等到一个 episode 结束才能进行计算。 TD是可以执行一次行为计算一次，不需要等到 episode 结束, 同时还具有更好的效率。

TD 的更新公式如下

$$
v(s) = v(s) + \alpha (R + \gamma v(s^\prime) - v(s))
$$ 

```python
v=[0.0, 0.1, 0.3, 0.5, 0.6, 0.8, 0.0]
gamma = 1.0
alpha = 0.001
for n in range(100000):
    s = 3
    while True:
        action = random.randint(0,2)
        s_next = s + 1
        if(action == 0):
            s_next = s - 1
        reward = 0.0
        if(s_next == 6):
            reward = 1.0 
        v[s] = v[s] + alpha * (reward + gamma * v[s_next] - v[s])
        if(s_next == 0 or s_next == 6):
            break
        s = s_next

print(v)
```


这个地方我们可以改变v的初值试试，比如改成 [0.0, 0.1, 0.3, 0.5, 0.6, 0.8, 1.0] 就会发现不收敛，因为TD是有偏置的,如果初值设置的很不好那么会得不到好的好的结果。

#### SARSA

SARSA 是一种基于 TD 的 control 的方法, 是典型的策略评估和策略提升的过程。通过$\epsilon-greedy$ 提升策略产生动作$a^\prime$, 然后使用如下公式进行策略评估。最终收敛到最优策略。

$$
Q(s,a) = Q(s,a) + \alpha (R + \gamma Q(s^\prime, a^\prime) - Q(s,a))
$$

```python
q={
    (0, 0): 0.0,
    (0, 1): 0.0,
    (1, 0): random.random(),
    (1, 1): random.random(),
    (2, 0): random.random(),
    (2, 1): random.random(),
    (3, 0): random.random(),
    (3, 1): random.random(),
    (4, 0): random.random(),
    (4, 1): random.random(),
    (5, 0): random.random(),
    (5, 1): random.random(),
    (6, 0): 0.0,
    (6, 1): 0.0,
}
gamma = 0.9
alpha = 0.001
epsilon = 0.05
for n in range(10000):
    s = 3
    a = 1
    while True:
        # epsilon greedy
        a_ = 1
        if(q[(s, 0)] > q[(s, 1)]):
            a_ = 0
        if(random.random() < epsilon):
            a_ = random.randint(0,2)
        # the next action
        s_ = s + 1
        if(a_ == 0):
            s_ = s - 1
        reward = 0.0
        if(s_ == 6):
            reward = 1.0 
        q[(s,a)] = q[(s,a)] + alpha * (reward + gamma * q[(s_,a_)] - q[(s,a)])
        if(s_ == 0 or s_ == 6):
            break
        s, a = s_, a_

print(q)
```

#### Q-Learning

Q-Learning 是另外一种基于 TD 学习最优策略的方法,  使用一种策略$\mu$ ($epsilon-greedy$)来产生行为并执行，然后使用另一种策略$\pi$ (greedy) 进行策略提升, 然后再进行策略评估, 直到收敛到最优值函数$q_{\*}$

值函数更新公式, 首先, 通过 greedy 的方式选出 action

$$
a^\prime=\mathop{argmax}_{a \in A}q(s^\prime, a)
$$

然后使用 TD 公式对策略进行评估

$$
\begin{align}
Q(s, a) &= Q(s, a) + \alpha(R + \gamma Q(s^\prime, a^\prime) - Q(s,a)) \\
&= Q(s, a) + \alpha( R + \gamma \mathop{max}_{a^\prime \in A}Q(s^\prime, a^\prime) - Q(s,a)) 
\end{align}
$$

```python
q={
    (0, 0): 0.0,
    (0, 1): 0.0,
    (1, 0): random.random(),
    (1, 1): random.random(),
    (2, 0): random.random(),
    (2, 1): random.random(),
    (3, 0): random.random(),
    (3, 1): random.random(),
    (4, 0): random.random(),
    (4, 1): random.random(),
    (5, 0): random.random(),
    (5, 1): random.random(),
    (6, 0): 0.0,
    (6, 1): 0.0,
}
gamma = 0.9
alpha = 0.001
epsilon = 0.05
for n in range(10000):
    s = 3
    while True:
        #print("s:%s" %s)
        # epsilon greedy
        a = 1
        if(q[(s, 0)] > q[(s, 1)]):
            a = 0
        if(random.random() < epsilon):
            a = random.randint(0,2)
        # the next action
        s_ = s + 1
        if(a == 0):
            s_ = s - 1
        reward = 0.0
        if(s_ == 6):
            reward = 1.0 
        q[(s,a)] = q[(s,a)] + alpha * (reward + gamma * max(q[(s_, 0)], q[(s_, 1)]) - q[(s,a)])
        if(s_ == 0 or s_ == 6):
            break
        s = s_
    #print("\n\n")

print(q)
```

#### TD($\lambda$)

这儿使用 back-froward 

$$
E_0(s) = 0
$$

$$
E_t(s) = \gamma \lambda E_{t-1}(s) + 1(S_t=s)
$$

$$
\delta_t= R_{t+1} + \gamma V(S_{t+1}) - V(S_t)
$$

$$
V(S_t) = V(S_t) + \alpha \delta_t E_t(S_t)
$$


```python
v=[0.0, 0.1, 0.3, 0.5, 0.6, 0.8, 0.0]
gamma = 1.0
alpha = 0.001
lamb = 0.9
e=[0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
for n in range(10000):
    s = 3
    while True:
        action = random.randint(0,2)
        s_next = s + 1
        if(action == 0):
            s_next = s - 1
        reward = 0.0
        if(s_next == 6):
            reward = 1.0 
        for i in range(1,6):
            e[i] = lamb*gamma*e[i]
        e[s] = e[s] + 1
        delta = reward + gamma * v[s_next] - v[s]
        v[s] = v[s] + alpha * delta * e[s]
        if(s_next == 0 or s_next == 6):
            break
        s = s_next

print(v)
```

#### SARSA($\lambda$)

使用TD($\lambda$)进行更新的SARSA就是SARSA($\lambda$)

$$
E_0(s,a) = 0
$$

$$
E_t(s,a) = E_{t-1}(s, a) + 1(s=S_t, a=A_t)
$$

$$
delta_t = R_{t+1} + \gamma Q(S_{t+1},A_{t+1}) -Q(S_t, A_t)  
$$

$$
Q(S_t, A_t) = Q(S_t, A_t) + \alpha \delta_t E_t(S_t, A_t)
$$

```python
q={
    (0, 0): 0.0,
    (0, 1): 0.0,
    (1, 0): random.random(),
    (1, 1): random.random(),
    (2, 0): random.random(),
    (2, 1): random.random(),
    (3, 0): random.random(),
    (3, 1): random.random(),
    (4, 0): random.random(),
    (4, 1): random.random(),
    (5, 0): random.random(),
    (5, 1): random.random(),
    (6, 0): 0.0,
    (6, 1): 0.0,
}
e={
    (0, 0): 0.0,
    (0, 1): 0.0,
    (1, 0): 0.0,
    (1, 1): 0.0,
    (2, 0): 0.0,
    (2, 1): 0.0,
    (3, 0): 0.0,
    (3, 1): 0.0,
    (4, 0): 0.0,
    (4, 1): 0.0,
    (5, 0): 0.0,
    (5, 1): 0.0,
    (6, 0): 0.0,
    (6, 1): 0.0,
}
gamma = 0.9
alpha = 0.001
epsilon = 0.05
lambd = 0.9
for n in range(10000):
    s = 3
    a = 1
    while True:
        # epsilon greedy
        a_ = 1
        if(q[(s, 0)] > q[(s, 1)]):
            a_ = 0
        if(random.random() < epsilon):
            a_ = random.randint(0,2)
        # the next action
        s_ = s + 1
        if(a_ == 0):
            s_ = s - 1
        reward = 0.0
        if(s_ == 6):
            reward = 1.0 
        for k, v in e.iteritems():
            e[k] = gamma * lambd * v
        e[(s,a)] = e[(s,a)] + 1
        delta = reward + gamma * q[(s_,a_)] - q[(s,a)]
        q[(s,a)] = q[(s,a)] + alpha * delta * e[(s, a)]
        if(s_ == 0 or s_ == 6):
            break
        s, a = s_, a_

print(q)
```