---
title: XGBoost 源码阅读gbm模块
date: 2018-01-04 23:15:04
categories: machine learning
tags: [xgboost, 机器学习, machine learning, tree, gbdt, gbm]
---

## GBLinear

### Coordinate Descent

Coordinate Descent 是一种优化算法，主要的思想是固定其他的坐标方向，先将其中一个左边方向优化到目标函数的最小值，然后再重新选取坐标轴，重复进行优化，直到优化到收敛，这种方法对于可导不可导的都可以进行优化。

### Stochastic Coordinate Descent

对于目标函数

$$
\mathop {min}_{w \in R^d} \frac{1}{m} \sum_{i=1}^m L(\langle w, x_i \rangle, y_i) + \lambda {||w||}_1
$$

将其中的内容简写一下

$$
C(w) \equiv \frac{1}{m} \sum_{i=1}^m L(\langle w, x_i \rangle, y_i)
$$

$$
P(w) \equiv \frac{1}{m} \sum_{i=1}^m L(\langle w, x_i \rangle, y_i) + \lambda {||w||}_1
$$

SCD 的其中一种实现如下

![](http://7q5fny.com1.z0.glb.clouddn.com/scd.jpg)

其中 $g_j=(\Delta C(w))_j$, $\beta$ 是loss的二阶导数的上界

![](http://7q5fny.com1.z0.glb.clouddn.com/shooting.png)

![](http://7q5fny.com1.z0.glb.clouddn.com/shotgun.png)



### Elastic Net 

#### 更新

elastice net 是同时使用 $L_1$ 范数和 $L_2$范数正则项的一种线性模型, xgboost里面的gblinear就是实现了使用坐标下降算法来优化的elastice net

$$
obj =  \sum_{i=1}^m l(\langle w, x_i \rangle, y_i) + \lambda_2 {||w||}_2 + \lambda_1 {||w||}_1
$$

其中

$$
\hat{y_i} = \langle w, x_i \rangle = w_0 + w_1 x_1 + w_2 x_2 + w_3 x_4 + .... 
$$

那么求一下其中的 $g_i$ 和 $\beta$

对于 $w_0$

$$
\frac{\partial obj}{\partial w_0} = \sum_{i=1}^m l^\prime(\hat{y_i}, y_i) + 2\lambda_2 w_0
$$

$$
\frac{\partial^2 obj}{\partial w_0^2} = \sum_{i=1}^m l^{\prime \prime}(\hat{y_i}, y_i) + 2\lambda_2
$$

对于 $w_j$

$$
\frac{\partial obj}{\partial w_j} = \sum_{i=1}^m l^\prime(\hat{y_i}, y_i) x_j + 2\lambda_2 w_j
$$

$$
\frac{\partial^2 obj}{\partial w_j^2} = \sum_{i=1}^m l^{\prime \prime}(\hat{y_i}, y_i) x_j^2  + 2\lambda_2
$$


根据之前的算法描述，每次迭代的更新值

对于 $w_0$

$$
\delta w_0 = - \frac{\sum_{i=1}^m l^\prime(\hat{y_i}, y_i) + 2\lambda_2 w_0}{\sum_{i=1}^m l^{\prime \prime}(\hat{y_i}, y_i) + 2\lambda_2}
$$

对于 $w_j$

当 $w_j > \frac{\sum_{i=1}^m l^\prime(\hat{y_i}, y_i) x_j + 2\lambda_2 w_j}{\sum_{i=1}^m l^{\prime \prime}(\hat{y_i}, y_i) x_j^2  + 2\lambda_2}$

$$
\delta w_j = - \frac{\sum_{i=1}^m l^\prime(\hat{y_i}, y_i) x_j + 2\lambda_2 w_j + \lambda_1}{\sum_{i=1}^m l^{\prime \prime}(\hat{y_i}, y_i) x_j^2  + 2\lambda_2}
$$

当小于

$$
\delta w_j = - \frac{\sum_{i=1}^m l^\prime(\hat{y_i}, y_i) x_j + 2\lambda_2 w_j - \lambda_1}{\sum_{i=1}^m l^{\prime \prime}(\hat{y_i}, y_i) x_j^2  + 2\lambda_2}
$$


描述一下使用shotgun 的elastic net, 为了和原文对照，使用跟代码里面相同的变量来描述

$L_2$ : reg_lambda

$L_1$ : reg_alpha

针对偏置的 $L_2$ : reg_lambda_bias

针对偏置的更新算法:

```
对所有样本:
    sum_grad: 样本的g的和
    sun_hess: 样本的h的和
delta_w = - (sum_grad + reg_lambda_bias * w ) / (sum_hess + reg_lambda_bias)
w = w + delta_w
对所有的样本:
    更新样本的梯度 g = h * dw
```

针对$w_j$的更新算法

```
对所有的样本
    sum_grad = sum_grad + g * v
    sum_hess = sum_hess + h * v *v
if(sum_hess < eps) 
    delta_w = 0
if(w > (sum_grad + reg_lambda * w)/ (sum_hess + reg_lambda))
    delta_w = max(-(sum_grad + reg_lambda*w + reg_alpha)/ (sum_hess + reg_lambda), -w)
else 
    delta_w = min(-(sum_grad + reg_lambda*w - reg_alpha)/ (sum_hess + reg_lambda), -w)
w = w + delta_w

对所有的样本
    更新 g = h * v * dw
```


#### 预测

由于是线性模型，预测很简单，就是把系数和特征值乘起来相加就行了


## GBTree

GBTree 在这一部分的代码很简单，根据数据集产生新的树加入模型中。

## Dart

Dart 是在GBTree的基础上增加了drop机制，在没每轮迭代的时候，会随机将一些树丢掉，然后使用剩下的树来生成新的树

假设在第m轮训练，选择了k颗树进行丢弃

$D$表示要丢掉的树

$$
D=\sum_{i \in K} F_i
$$

$F_m$ 表示新增加的树

$$
F_m=\eta \tilde{F_m}
$$

目标函数可以写作

$$
obj=\sum_{j=1}^{n}L(y_j, \hat{y}_{j}^{m-1}-D_j+\tilde{F_m}) + \Omega(\tilde{F_m})
$$

由于系数的不一致，需要对系数进行调整

$$
\hat{y}_j^m=\sum_{i \notin K}F_i + a(\sum_{i \in k} F_i + bF_m)
$$

归一化

有两种归一化的方式，通过参数可以进行设置

tree: 新增加的树的权重与被丢弃的每一颗树的权重相等

$b$取$\frac{1}{k}$

$$
a(\sum_{i \in k} F_i + \frac{1}{k}F_m) = a(\sum_{i \in k} F_i + \frac{\eta}{k}\tilde{F}_m)
$$

由于 $\tilde{F}_m$ 是在去掉$D$的基础上学出来的，那么 $\tilde{F}_m$ 的值应该是和$D$差不多的, 同时在将新的树和丢掉的树放回去的时候，应该保持权重也基本为$D$, 可以得到

$$
a(1+\frac{\eta}{k})D = D
$$

可以得到 

$$
a=\frac{k}{k+\eta}
$$


forest: 新增加的树的权重与被丢掉的树的和相等

同样可以得到

$$
a=\frac{1}{1+\eta}
$$

