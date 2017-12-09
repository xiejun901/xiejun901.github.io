title: xgboost 学习笔记 - 原理简介
date: 2017-11-30 23:44:53
categories: machine learning
tags: [xgboost, 机器学习, machine learning, tree, gbdt]
---


可以通过再树的叶子节点上增加权重来使用决策树做回归问题，同时，为了提高结果的准确度，可以使用多颗树来构建模型对样本进行预测。
假设每棵树使用 $f_k$ 来表示，那么一个使用 $K$ 颗树的模型可以使用如下方式来表示预测结果

$$
\hat{y}_i=\sum_{k=1}^Kf_k(x_i)
$$

$$
\theta
$$
