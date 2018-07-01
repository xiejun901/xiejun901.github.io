---
title: 反向传播
date: 2018-06-11 22:50:54
tags: [反向传播, 机器学习, 神经网络, 梯度下降, 求导, 深度学习]
categories: machine learning
---

在机器学习中，我们通常是使用梯度下降来更新参数，来使得我们的损失函数最小。

$$
\theta = \theta -  \eta*\nabla_{\theta}J(\theta)
$$

其中求梯度是非常重要的一个步骤，而通常所说的反向传播就是一种常见的求梯度方法。

## 常见的求梯度方法

### 符号求导

可以将带符号的表达式表示成一颗表达式树，然后利用常见的求导规则，可以进行化简和求导

### 数值求导

利用如下公式进行求导

$$
\frac{\partial f(x)}{\partial x_i} \approx \mathop {limit}_{h->0} \frac{f(x+h e_i) - f(x)}{h}
$$

或者

$$
\frac{\partial f(x)}{\partial x_i} \approx \mathop {limit}_{h->0} \frac{f(x+h e_i) - f(x-h e_i)}{2h}
$$

### 反向传播

利用链式法则的求导方式

$$
y=f(x)
$$

$$
z=g(y)
$$

$$
\frac{\partial z}{\partial x} = \frac{\partial z}{\partial y}  \frac{\partial y}{\partial x} = g'(x)f'(x) 
$$

反向传播则是将计算先抽象乘运算符和变量，构造成计算图，然后根据每个变量的值和运算符的求导运算来计算图中任意两个变量的偏导数。后边会详细介绍算法和做一个简单的实现

### 自动求导

自动求导和反向传播的原理是一样的，可以认为是反向传播的不同的实现方式，在构造好计算图之后，根据要求的导数，直接生成新的节点，然后在计算导数的时候就跟前向运算一样，直接通过计算图计算就行了。好处是可以很方便的进行系统优化。tensorflow 和 Theano 都是使用的这种方式


## 反向传播

[cs231n 反向传播](http://cs231n.github.io/optimization-2/) 这儿有一篇非常详细的反向传播原理的介绍。 下面介绍下代码的实现

[自己实现的完整代码](https://github.com/xiejun901/ToyDL/blob/master/python/back_prop.py)

### 节点和运算符的表示

需要定义最基本的两个数据结构变量节点和运算符。
对于每个节点，我们需要保存产生此变量的运算符和此变量是由哪些节点计算而来，同时为了处理字面值的常量和节点的运算，使用了一个成员变量来表示这个常量。对于运算符，我们需要定义一个前向运算和反向传播运算。前向运算用来计算变量节点的值，反向传播运算用来求导

```python
class Node(object):

    def __init__(self):
        self.inputs = []
        self.outputs = []
        self.op = None
        self.name = ""
        self.const_attr = None

    def __add__(self, other):
        if isinstance(other, Node):
            return add_op(self, other)
        else:
            return add_byconst_op(self, other)

    def __mul__(self, other):
        if isinstance(other, Node):
            return mul_op(self, other)
        else:
            return mul_byconst_op(self, other)

    __radd__ = __add__
    __rmul__ = __mul__

    def __str__(self):
        return self.name

    __repr__ = __str__


class Op(object):

    def __call__(self):
        """
        调用运算符构造一个节点
        :return:
        """
        node = Node()
        node.op = self
        return node

    def compute(self, node, input_vals):
        """
        前向计算，根据输入的值计算出经过此运算符输出的值
        :param input_vals: 输入值 numpy.array
        :return:
        """
        raise NotImplementedError

    def bprop(self, node, input_vals, output_grad):
        """
        反向传播，根据输出的梯度和输入的值计算出输入的梯度
        :param input_vals:
        :param output_grad:
        :return:
        """
        raise NotImplementedError
```

### 前向计算和反向传播

Executor类中定义了前向计算和反向传播的方法，对于前向计算，我们只需要在计算出节点的依赖节点的值之后使用节点中存储的运算符计算节点的值就行了。对于反向传播，计算对于某个节点的导数的时候，需要计算好输入的值，然后根据后继节点的导数，调用运算符的bprop方法就可以实现。具体代码如下

```python
class Executor(object):

    def __init__(self):
        self.node_to_val_map = {}
        self.node_to_grad_map = {}

    def forward(self, eval_node_list , feed_dict):
        self.node_to_val_map = dict(feed_dict)
        for node in eval_node_list:
            self.eval(node)
        return [self.node_to_val_map[n] for n in eval_node_list]

    def backward(self, output_node, grad_node_list, feed_dict):
        self.node_to_val_map = dict(feed_dict)
        self.node_to_grad_map = {}
        self.eval(output_node) # 计算输出节点及其依赖的节点的值
        self.node_to_grad_map[output_node] = np.ones_like(self.node_to_val_map[output_node])
        for node in grad_node_list:
            self.eval_grad(node)
        return [self.node_to_grad_map[n] for n in grad_node_list]

    def eval(self, node):
        if node in self.node_to_val_map:
            return
        for input_node in node.inputs:
            self.eval(input_node)
        input_vals = [self.node_to_val_map[input] for input in node.inputs]
        self.node_to_val_map[node] = node.op.compute(node, input_vals)

    def eval_grad(self, node):
        from functools import reduce
        if node in self.node_to_grad_map:
            return
        temp = []
        for output in node.outputs:
            if output not in self.node_to_val_map:
                """
                需要计算梯度的是被计算点的后续节点，和输出点的祖先节点，由于事先计算过输出节点的值，如果在值table里面不存在，则不是输出
                节点的祖先节点
                """
                continue
            self.eval_grad(output)
            input_vals = [self.node_to_val_map[n] for n in output.inputs]
            output_grad = self.node_to_grad_map[output]
            input_grads = output.op.bprop(output, input_vals, output_grad)
            for i in range(len(output.inputs)):
                if output.inputs[i] == node:
                    temp.append(input_grads[i])
        grad = reduce(lambda x1, x2: x1 + x2, temp)
        self.node_to_grad_map[node] = grad

```

### 常见运算符的实现

在反响传播中，实际上求导都是最终的损失函数(标量)对中间变量求导，因此可以使用标量对向量的求导来理解

在这儿求导的推导可以参考这篇文章 [矩阵求导术（上）](https://zhuanlan.zhihu.com/p/24709748) 文章中通过微分方程来推导标量对矩阵的求导。

主要使用以下几条规则来进行矩阵微分运算和迹技巧

#### 导数与微分方程关系

$$
df=\sum_{i=1}^{m} \sum_{j=1}^{n} \frac{\partial f}{\partial X_{ij}}dX_{ij}=tr(\frac{\partial f}{\partial X}^T dX)
$$

#### 矩阵微分运算法则

1. $d(X \pm Y) = dX \pm dY$ , $d(XY) = dXY + XdY$ , $d(X^T) = (dX)^T$ , $dtr(X) = tr(dX)$
2. $d(X^{-1})=-X^{-1}dXX^{-1}$ 
3. $d|X| = tr(X^\\#dX)$ 当X可逆的时候可写作 $d|X|=|X|tr(X^{-1}dX)$
4. $d(X\bigodot Y)=dX \bigodot Y+X \bigodot dY$
5. $d\sigma(X)=\sigma ^ \prime (X) \bigodot dX$

#### 迹技巧

1. 标量套上迹 $a=tr(a)$
2. 转置 $tr(A^T)=tr(A)$
3. 线性 $tr(A \pm B) = tr(A) \pm tr(B)$
4. 矩阵乘法交换  $tr(AB)=tr(BA)$
5. 矩阵乘法/逐元素乘法交换 $ tr(A^T(B \bigodot C)) =  tr((A \bigodot B)^TC)$

通过以上规则就能进行求导推导, 注意，这个地方没有直接定义定义矩阵对矩阵求导，因此对于复合函数求导不能直接沿用链式法则，需要通过微分代换的方式进行计算。

#### Placeholder

这个运算主要是为了产生输入的，此节点的值由feed_dict 提供，不应该在此节点上调用前向运算，对它的导数也是由它的后继节点上的运算符来计算出的。

```python
class PlaceholderOp(Op):

    def __call__(self, *args, **kwargs):
        node = Op.__call__(self)
        return node

    def compute(self, node, input_vals):
        assert False, "placeholder values provided by feed_dict"

    def bprop(self, node, input_vals, output_grad):
        assert False, "can not do back propagation"
```


#### 加法

对于加法运算，输出对两个输入的导数就是输出本身

![](adddiff.png)

```python
class AddOp(Op):

    def __call__(self, node1, node2):
        node = Node()
        node.inputs = [node1, node2]
        node.op = self
        node.name = "{}+{}".format(node1.name, node2.name)
        node1.outputs.append(node)
        node2.outputs.append(node)
        return node

    def compute(self, node, input_vals):
        assert len(input_vals) == 2
        return input_vals[0] + input_vals[1]

    def bprop(self, node ,input_vals, output_grad):
        return [output_grad, output_grad]
```

#### 矩阵乘法

![](matmuldiff.png)


```python
class MatMulOp(Op):

    def __call__(self, node1, node2):
        node = Node()
        node.inputs = [node1, node2]
        node.op = self
        node.name = "MatMulOp({},{})".format(node1.name, node2.name)
        node1.outputs.append(node)
        node2.outputs.append(node)
        return node

    def compute(self, node, input_vals):
        assert len(input_vals) == 2
        return np.matmul(input_vals[0], input_vals[1])

    def bprop(self, node, input_vals, output_grad):
        return [np.matmul(output_grad,np.transpose(input_vals[1])), np.matmul(np.transpose(input_vals[0]), output_grad)]
```

#### 逐元素乘法

![](muldiff.png)

```python
class MulOp(Op):

    def __call__(self, node1, node2):
        node = Node()
        node.inputs = [node1, node2]
        node.op = self
        node.name = "{}*{}".format(node1.name, node2.name)
        node1.outputs.append(node)
        node2.outputs.append(node)
        return node

    def compute(self, node, input_vals):
        assert len(input_vals) == 2
        return input_vals[0] * input_vals[1]

    def bprop(self, node, input_vals, output_grad):
        assert len(input_vals) == 2
        return [input_vals[1]*output_grad, input_vals[0]*output_grad]
```


#### 逐元素函数

这儿以sigmoid为例

![](fdiff.png)

```python
class SigmoidOp(Op):

    def __call__(self, node1):
        node = Node()
        node.inputs = [node1]
        node.op = self
        node.name = "Sigmoid({})".format(node1.name)
        node1.outputs.append(node)
        return node

    def compute(self, node, input_vals):
        assert len(input_vals) == 1
        return sigmoid_func(input_vals[0])

    def bprop(self, node, input_vals, output_grad):
        temp = sigmoid_func(input_vals[0])
        return [output_grad * (1 - temp) * temp]
```


#### 其他

其他的运算符的实现基本与上面一致，推导也差别不大。实现的源码在 [反向传播](https://github.com/xiejun901/ToyDL/blob/master/python/back_prop.py)
