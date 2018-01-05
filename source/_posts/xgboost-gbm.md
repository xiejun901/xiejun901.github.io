---
title: XGBoost 源码阅读gbm模块
date: 2018-01-04 23:15:04
categories: machine learning
tags: [xgboost, 机器学习, machine learning, tree, gbdt, gbm]
---

## GBLinear

### Coordinate Descent

Coordinate Descent 是一种优化算法，主要的思想是固定其他的坐标方向，先将其中一个左边方向优化到目标函数的最小值，然后再重新选取坐标轴，重复进行优化，直到优化到收敛，这种方法对于可导不可导的都可以进行优化。

对于自变量

$$
x^0=(x_1^0, x_n^0, ... ,x_n^0)
$$

### Elastic Net 


## GBTree

## Dart