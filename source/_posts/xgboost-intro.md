title: xgboost 学习笔记 - 原理简介
date: 2017-11-30 23:44:53
categories: machine learning
tags: [xgboost, 机器学习, machine learning, tree, gbdt]
---

可以通过在树的叶子节点上增加权重来使用决策树做回归问题，同时，为了提高结果的准确度，可以使用多颗树来构建模型对样本进行预测。
假设每棵树使用 $f_k$ 来表示，那么一个使用 $K$ 颗树的模型可以使用如下方式来表示预测结果

$$
\hat{y}_i=\sum_{k=1}^Kf_k(x_i)
$$

根据以上公式

$$
\hat{y_i}^{(0)} = 0
$$

$$
\hat{y_i}^{(1)} = \hat{y_i}^{(0)} + f_1(x_i)
$$

$$
\hat{y_i}^{(2)} = \hat{y_i}^{(1)} + f_2(x_i)
$$

$$
\hat{y_i}^{(3)} = \hat{y_i}^{(2)} + f_3(x_i)
$$

$$
......
$$

$$
\hat{y_i}^{(t)} = \hat{y_i}^{(t-1)} + f_t(x_i)
$$

也就是说第前$t$颗树的预测结果可以表示为前$t-1$颗树的预测结果与第$t$颗树的结果的和。也就是说我们可以从第1颗树开始，挨着训练每一棵树来使得模型更准确。
模型的准确性是通过目标函数来评估的，训练的目标就是让在所有样本上，目标函数要尽量小。
我们使用$l(y_i, \hat{y_i})$ 来表示对于输入样本$i$的labe $y_i$ 和预测值$\hat{y_i}$的误差

那么在第$t$轮训练的时候，我们的目标函数为

$$
obj^{(t)} = \sum_{i=1}^n l(y_i, \hat{y_i}^{(t)}) + \sum_{j=1}^t\Omega(f_j) \\
=\sum_{i=1}^n l(y_i, \hat{y_i}^{(t-1)} + f_t(x_i)) + \sum_{j=1}^t\Omega(f_j)
$$

根据泰勒展开公式

$$
f(x+\Delta x) = \sum_{j=0}^\infty{\frac{\Delta x ^j}{j!}}f^{(n)}(x)
$$
 
$$
g_i = \partial_{\hat{y_i}^{(t-1)}} l(y_i, \hat{y_i}^{(t-1)})
$$

