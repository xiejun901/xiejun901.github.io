title: sgd 中的优化方法
date: 2017-07-03 00:08:38
tags: [sgd, machine learning]
categories: machine learning
---

最近在看 [fast.ai 上的深度学习教程](http://course.fast.ai/) ，这儿对学习到的内容做一个笔记。结合网上找到的相关资料对sgd中的优化算法做一个记录。具体的代码使用两个参数的线性回归作为例子: y = a*x + b

### 示例代码数据准备

使用的是线性回归，参数有 a 和 b, 使用平方损失函数

$$
\begin{align}
L &= (y^\prime - y)^2 \\
&=(ax + b -y)^2
\end{align}
$$

$$
\frac{ \partial L}{\partial a} = 2x(ax+b-y)
$$

$$
\frac{ \partial L}{\partial b} = 2(ax+b-y)
$$



```python
import numpyt as np
import numpy as np
from matplotlib import pyplot as plt
import math

a = 2.0
b = 30.0
x = np.random.randint(0, 100, 30)
y = a * x + b
```

### 梯度下降

#### Batch gradient descent

最基本的梯度下降是 Batch gradient descent, 一次使用所有的数据进行更新, 对于损失函数$L(\theta)$, 参数更新的方式为

$$
\theta = \theta -  \eta*\nabla_{\theta}J(\theta)
$$

```python
x = np.random.randint(0, 100, 30)
a = 2.
b = 30.
y = a* x +b
a_guess = 1.0
b_guess = 1.0
eta = 0.000001
def fit(x, y, epochs):
    global a_guess, b_guess
    for i in range(epochs):
        t = (x*a_guess+b_guess - y)
        a_guess = a_guess - eta*2*t.dot(x)
        b_guess = b_guess - eta*2*t.sum()
        if i %50000 == 0:
            print("epoch %d, a=%g, b=%g" %(i, a_guess, b_guess))

fit(x, y, 500000)
```

要跑很久很久参数才能大概收敛，Batch gradient descent 是非常慢的，并且当训练数据过大，无法同时放入内存中，也没法通过Batch gradient descent进行计算。

#### Stochastic gradient descent

sgd 是在每次使用一个样本进行梯度更新，通常来讲sgd 会比批量梯度下降算法更快，并且不一定要将数据一次读入内存中。

$$
\theta = \theta -  \eta*\nabla_{\theta}J(\theta, x^{(j)}, y^{(j)})
$$

```python
x = np.random.randint(0, 100, 30)
a = 2.
b = 30.
y = a* x +b
a_guess = 1.0
b_guess = 1.0
eta = 0.0001
def fit(x, y, epochs):
    global a_guess, b_guess
    for i in range(epochs):
        sample = np.array((x, y)).T
        np.random.shuffle(sample)
        for j in range(len(x)):
            x_, y_ = sample[j][0], sample[j][1]
            t = (a_guess * x_ + b_guess - y_)
            a_guess = a_guess - eta*2*t*x_
            b_guess = b_guess - eta*2*t
        if i %1000 == 0:
            print("epoch %d, a=%g, b=%g" %(i, a_guess, b_guess))

fit(x, y, 10000)
```

的确比批量梯度下降快了很多

#### Mini-batch gradient descent

Mini-batch gradient descent 结合了批量梯度下降和sgd的优点，每次更新的时候使用一部分数据来计算更新量，通常batch size 的选择范围是 50-256，Mini-batch gradient descent 具有以下优点

- 降低了更新参数的方差，使得收敛更稳定
- 可以通过矩阵运算来进行优化

更新的公式为

$$
\theta = \theta -  \eta*\nabla_{\theta}J(\theta, x^{(j:j+n)}, y^{(j:j+n)})
$$

```python
x = np.random.randint(0, 100, 30)
a = 2.
b = 30.
y = a* x +b
a_guess = 1.0
b_guess = 1.0
eta = 0.00001
def fit(x, y, epochs):
    global a_guess, b_guess
    for i in range(epochs):
        sample = np.array((x, y)).T
        np.random.shuffle(sample)
        batches = get_batches(sample)
        for batch in batches:
            x_, y_ = batch[:, 0], batch[:, 1]
            t = (a_guess * x_ + b_guess - y_)
            a_guess = a_guess - eta*2*t.dot(x_)
            b_guess = b_guess - eta*2*t.sum()
        if i %5000 == 0:
            print("epoch %d, a=%g, b=%g" %(i, a_guess, b_guess))

def get_batches(sample, batch_size = 10):
    batches = sample[0:batch_size*len(sample)/batch_size].reshape(len(sample)/batch_size, batch_size, -1)
    return batches
        
fit(x, y, 50000)
```

### 梯度下降中的优化算法

为了加快梯度下降算法的收敛速度和稳定性，人们发明了很多优化算法。

#### momentum

momentum 是动量的意思，在进行梯度更新的时候不止考虑当前batch或者样本算出来的更新量，还考虑之前样本算出来的更新量，就像小球从山上滚下来的时候，不只是按照当前最陡的方向滚动，还因为惯性的原因，在原来的方向上也有一个分量。在momentum中，是对之前的梯度进行了一个指数衰减。

$$
v = \gamma \* v + \eta \* \nabla_{\theta}J(\theta)
$$


$$
\theta = \theta -  v
$$

一般来说$\gamma$的取值会先取0.5，然后修改为0.9或者0.99

```python
x = np.random.randint(0, 100, 30)
a = 2.
b = 30.
y = a* x +b
a_guess = 1.0
b_guess = 1.0
eta = 0.00001
gamma = 0.9
v_a=0.0
v_b=0.0
def fit(x, y, epochs):
    global a_guess, b_guess, v_a, v_b
    for i in range(epochs):
        sample = np.array((x, y)).T
        np.random.shuffle(sample)
        for j in range(len(x)):
            x_, y_ = sample[j][0], sample[j][1]
            t = (a_guess * x_ + b_guess - y_)
            v_a = gamma * v_a + eta * 2 * t * x_
            v_b = gamma * v_b + eta * 2 * t
            a_guess = a_guess - v_a
            b_guess = b_guess - v_b
        if i %1000 == 0:
            print("epoch %d, a=%g, b=%g" %(i, a_guess, b_guess))

fit(x, y, 10000)
```

在 [fast.ai的课程上](http://course.fast.ai/) , momentum的公式写成了以下形式，其实是没有区别的，只是参数的具体大小会有一些调整

$$
v = \beta_0 \* v + \beta_1 \* \nabla_{\theta}J(\theta)
$$

$$
\theta = \theta -  \eta \* v
$$

#### adagrad

在momentum中，对于所有的参数都使用相同的更新速率，当参数的大小相差特别大的时候，就会出现有的参数已经更新到很好的值了，但是另外的参数还差很远，并且只能以较小的更新速率慢慢更新，会严重影响参数的收敛速度。adgrad就是对于不同参数，引入了不同的更新速率

对于第t个样本(批次) 使用 $g(t, i)$来表示对参数$\theta_i$计算出来的梯度

$$
g(t,i)=\nabla_{\theta}J(\theta_i)
$$

sgd 的更新方式是

$$
\theta_{t+1,i}  =  \theta_{t,i} - \eta*g(t,i)
$$

在adagrad中引入一个新的变量来改变不同参数的更新速率

$$
\theta_{t+1,i}  =  \theta_{t,i} - \frac{\eta}{\sqrt{G_{t,ii} + \epsilon}} \* g(t,i)
$$

$\epsilon$ 使一个极小量来防止除以0的。$G_{t,ii}$ 是对于参数$\theta_i$,过去所有的梯度的平方和

$$
G_(t, ii) = \sum_{k=0}^{i}g(k,i)
$$

```python
x = np.random.randint(0, 100, 30)
a = 2.
b = 30.
y = a* x + b
a_guess = 1.
b_guess = 1.
a_g = 1918.
b_g =28.
eta = 0.1
epsilon=1e-8
def fit(x, y, epochs):
    global a_guess, b_guess, a_g, b_g
    for i in range(epochs):
        sample = np.array((x, y)).T
        np.random.shuffle(sample)
        for j in range(len(x)):
            x_, y_ = sample[j][0], sample[j][1]
            t = (a_guess * x_ + b_guess - y_)
            a_g = a_g + (2*t*x_) ** 2
            b_g = b_g + (2*t) ** 2
            a_guess = a_guess - eta / math.sqrt(a_g + epsilon) * 2 * t * x_
            b_guess = b_guess - eta / math.sqrt(b_g + epsilon) * 2 * t
        if i %5000 == 0:
            print("epoch %d, a=%g, b=%g, a_g=%g, b_g=%g" %(i, a_guess, b_guess, a_g, b_g))

fit(x, y, 100000)
```

adagrad 有个很大的问题就是随着迭代的次数增加，g 会越来越大，导致实际上的学习率就会趋向于无穷小，就没法再学习了
同时，g的初值对结果影响也还挺大的，学习率除以初值过小可能导致收敛时间过长甚至没法收敛；过大可能导致震荡，使得参数一下就变化巨大，然后学习率迅速变小导致也没法收敛到正确值。

#### rmsprop

rmspop 是 Geoff Hinton 在coursera的课程上提出的方法，[课程slides](http://www.cs.toronto.edu/~tijmen/csc321/slides/lecture_slides_lec6.pdf).

rmsprop对过去所有的梯度的平方和进行了加权，越近的权重越大，对历史的梯度平方作了指数衰减，使得在靠近极值的时候能更稳定

$$
meansquare(\theta, t) = \beta_1 \* meansquare(\theta, t-1) + \beta_2 \* (\nabla_{\theta}J(\theta))^2
$$

$$
\theta = \theta - \frac{\eta}{\sqrt{meansquare(\theta, t)}} \* \nabla_{\theta}J(\theta)
$$

$\beta_1$取值一般为0.9, $\beta_2$ 取值为0.1

```python
x = np.random.randint(0, 100, 30)
a = 2.
b = 30.
y = a* x +b
a_guess = 1.0
b_guess = 1.0
eta = 0.01
beta1 = 0.9
beta2 = 0.1
meansquare_a = 1.
meansquare_b = 1.
epsilon=1e-8
def fit(x, y, epochs):
    global a_guess, b_guess, meansquare_a, meansquare_b
    for i in range(epochs):
        sample = np.array((x, y)).T
        np.random.shuffle(sample)
        for j in range(len(x)):
            x_, y_ = sample[j][0], sample[j][1]
            t = (a_guess * x_ + b_guess - y_)
            meansquare_a = beta1 * meansquare_a + beta2 * (2 * t * x_)**2
            meansquare_b = beta1 * meansquare_b + beta2 * (2 * t)**2
            a_guess = a_guess - eta / math.sqrt(meansquare_a + epsilon) * 2 * t * x_
            b_guess = b_guess - eta / math.sqrt(meansquare_b + epsilon) * 2 * t
        if i %1000 == 0:
            print("epoch %d, a=%g, b=%g, meansquare_a=%g, meansquare_b=%g" %(i, a_guess, b_guess, meansquare_a, meansquare_b))

fit(x, y, 10000)
```

运行一下可以发现，到收敛的时候参数是不会溢出的，会在最优值附近变化，

#### adam

adam 是结合了momentum的一种优化方法

$$
momentum(\theta, t) = \beta_1 \* momentum(\theta, t-1) + (1 - \beta_1) \* \nabla_{\theta}J(\theta)
$$

$$
meansquare(\theta, t) = \beta_2 \* meansquare(\theta, t-1) + (1 - \beta_2) \* (\nabla_{\theta}J(\theta))^2
$$

$$
\theta = \theta - \frac{\eta}{\sqrt(meansquare(\theta, t))} \* momentum(\theta, t)
$$

```python
x = np.random.randint(0, 100, 30)
a = 2.
b = 30.
y = a* x +b
a_guess = 1.0
b_guess = 1.0
eta = 0.002
beta1 = 0.9
beta2 = 0.999
meansquare_a = 1.
meansquare_b = 1.
mom_a = 1.
mom_b = 1.
epsilon=1e-8
def fit(x, y, epochs):
    global a_guess, b_guess, meansquare_a, meansquare_b, mom_a, mom_b
    for i in range(epochs):
        sample = np.array((x, y)).T
        np.random.shuffle(sample)
        for j in range(len(x)):
            x_, y_ = sample[j][0], sample[j][1]
            t = (a_guess * x_ + b_guess - y_)
            mom_a = beta1 * mom_a + (1-beta1) * (2 * t * x_)
            mom_b = beta1 * mom_b + (1-beta1) * (2 * t)
            meansquare_a = beta2 * meansquare_a + (1-beta2) * (2 * t * x_)**2
            meansquare_b = beta2 * meansquare_b + (1-beta2) * (2 * t)**2
            a_guess = a_guess - eta / math.sqrt(meansquare_a + epsilon) * mom_a
            b_guess = b_guess - eta / math.sqrt(meansquare_b + epsilon) * mom_b
            
        if i %1000 == 0:
            print("epoch %d, a=%g, b=%g, meansquare_a=%g, meansquare_b=%g" %(i, a_guess, b_guess, meansquare_a, meansquare_b))

fit(x, y, 10000)
```


#### eve

eve 是在 adam 的基础上引入了自动退火机制，在使用adam或者之前那些动态学习率方法的时候，还是需要手动的去调节学习率，在最初的时候使用较大的学习率进行学习，学习一段时间后调小，反复进行这样的操作直到对结果满意。

$$
loss(t) = \sqrt{\sum(y - y^{\prime})^2}
$$

$$
r(t) = abs(loss(t) - loss(t-1)) / min(loss(t) - loss(t-1))
$$

$$
d(t) = \beta_3 \* d(t-1) + (1 - \beta_3) * r(t)
$$

$$
momentum(\theta, t) = \beta_1 \* momentum(\theta, t-1) + (1 - \beta_1) \* \nabla_{\theta}J(\theta)
$$

$$
meansquare(\theta, t) = \beta_2 \* meansquare(\theta, t-1) + (1 - \beta_2) \* (\nabla_{\theta}J(\theta))^2
$$

$$
\theta = \theta - \frac{\eta}{\sqrt{(meansquare(\theta, t)} \* d(t))} \* momentum(\theta, t)
$$




```python
x = np.random.randint(0, 100, 30)
a = 2.
b = 30.
eta = 1.
y = a* x +b
a_guess = 1.0
b_guess = 1.0
beta1 = 0.9 # for momentum
beta2 = 0.999 
meansquare_a = 1.
meansquare_b = 1.
mom_a = 1.
mom_b = 1.
epsilon=1e-8
ft_1 = 1.
ft_2 = 1.
r_t = 1.0
dt = 1.0
beta3 = 0.9
def fit(x, y, epochs):
    global a_guess, b_guess, meansquare_a, meansquare_b, mom_a, mom_b
    global ft_1, ft_2, r_t, dt
    for i in range(epochs):
        sample = np.array((x, y)).T
        np.random.shuffle(sample)
        for j in range(len(x)):
            x_, y_ = sample[j][0], sample[j][1]
            t = (a_guess * x_ + b_guess - y_)
            #loss = loss + (t ** 2)
            loss = math.sqrt(((a_guess * sample[:,0] + b_guess - sample[:, 1])**2).sum())
            ft_2 = ft_1
            ft_1 = loss
            r_t = abs(ft_2 - ft_1) / (min(ft_2, ft_1) + epsilon)
            dt = beta3*dt + (1-beta3)*r_t
            mom_a = beta1 * mom_a + (1-beta1) * (2 * t * x_)
            mom_b = beta1 * mom_b + (1-beta1) * (2 * t)
            meansquare_a = beta2 * meansquare_a + (1-beta2) * (2 * t * x_)**2
            meansquare_b = beta2 * meansquare_b + (1-beta2) * (2 * t)**2
            a_guess = a_guess - eta / math.sqrt(meansquare_a + epsilon) / dt * mom_a
            b_guess = b_guess - eta / math.sqrt(meansquare_b + epsilon) / dt * mom_b
            
        if i %5000 == 0:
            print("epoch %d, a=%g, b=%g, meansquare_a=%g, meansquare_b=%g" %(i, a_guess, b_guess, meansquare_a, meansquare_b))
            #print("          loss = %g, ft_1 = %g, ft_2 = %g, r_t = %g, dt=%g" %(loss, ft_1, ft_2, r_t, dt))

fit(x, y, 50000)
```

在eve中，当在计算结果很接近真实值的时候，loss变化会非常小，此时算法会增大学习率，使得参数的值偏离真实值，然后会再次靠近真实值，会产生波动。


