title: 使用tensorflow和mnist数据集训练CNN
date: 2017-07-18 23:33:23
tags: [mnist, cnn, machine learning, tensorflow]
categories: machine learning
---

本文是 [fast.ai 上的深度学习教程](http://course.fast.ai/) 的学习笔记，在课程上，讲到了使用 mnist 数据集训练 CNN 的例子，课程上使用的是 keras 来实现，为了加强理解和熟悉 tensorflow， 此处使用tensorflow 实现一遍。

### 获取数据集

tensorflow 直接提供了 mnist的数据集，只需要导入example中的包就行，然后获取数据的时候会自动检测指定目录下是否存在数据，不存在则会自动从网上下载

```python
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets('MNIST_DATA/', one_hot=True)
```

获取到的数据包含了训练集，测试集和验证集合, 其中训练集的大小为 55000 张, 测试集 10000 张, 验证集 5000 张. 每张图片的大小事28*28=784, 可以通过以下指令查看数据集的大小

```python
mnist.train.images.shape
```

可以得到结果是 (55000, 784)

可以将图片画出来看看

```python
from matplotlib import pyplot as plt
plt.gray()
def plot(img):
    plt.imshow(img.reshape((28, 28)))

plot(mnist.train.images[1])
```

![](http://7q5fny.com1.z0.glb.clouddn.com/number3.png)

查看一下label

```python
mnist.train.labels[1]
array([ 0.,  0.,  0.,  1.,  0.,  0.,  0.,  0.,  0.,  0.])
```

### 建立模型

先写两个工具函数建立指定的大小的便令，用作卷积filter和偏置

```
def weight_variable(shape):
    initial = tf.truncated_normal(shape, stddev=0.1)
    return tf.Variable(initial)
def bias_variable(shape):
    initial = tf.constant(0.1, shape=shape)
    return tf.Variable(initial)
```

在tensorflow中，Variable 是变量，用来存储可以修改的值, 例如模型参数，在 weight_variable 中，按照正态分布初始化一个指定形状的张量用来作卷积的参数。bias_variable 是初始化一个指定形状为常数的张量来作为偏置项。

可以通过如下方式来查看一下这两个函数的返回值

```python
sess = tf.Session()
conv = weight_variable([3, 3])
bias = bias_variable([5])
sess.run(tf.global_variables_initializer())
print(sess.run(conv))
print(sess.run(bias))
```

    [[-0.06172315  0.12105345  0.01371539]
     [ 0.10928184 -0.03302958 -0.12141398]
     [ 0.07855338  0.05833388  0.02793374]]
    [ 0.1  0.1  0.1  0.1  0.1]

图片的输入格式是55000张 1 \* 584 的数组，首先需要对输入进行reshape, 好用于做二维的卷积运算。将每张图片输入的584维 reshape 成 28 \* 28 \* 1 最后的1表示一个通道，可以理解为这个地方是黑白图片，所以只需要一维，如果是彩色图片的话就可能是3维。总之就是每张图片变成了一个 28 \* 28 \* 1的张量。

```
x = tf.placeholder(tf.float32, [None, 584])
x_images = tf.reshape(x, [-1, 28, 28, 1])
```

placeholder 在 tensorflow 中叫做占位符，用来接受输入数据。可以通过以下方式将输入传入网络中并进行计算

```python
sess.run(x_images, {x: train_imgs[0:1]})
```

在CNN中，一般会对输入的对象先进行卷积运算，提取图像的特征。

这儿再引入两个函数

```
def conv2d(x, W):
    return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')
def max_pool_2x2(x):
    return tf.nn.max_pool(x, ksize=[1, 2, 2, 1], strides=[1, 2, 2, 1], padding='SAME')
```

conv2d 使用来做2D卷积运算的, max_pool_2x2 是用来进行一个 2*2 的maxpolling, 图像的大小会减半。

接下来就可以开始建立我们的神经网络了, 本例中使用两层卷积网络加两层全连接层

第一层卷积层

```python
conv1_w = weight_variable([3, 3, 1, 32])
conv1_b = bias_variable([32])
h_conv1 = tf.nn.relu(conv2d(x_images, conv1_w)+conv1_b)
h_pool1 = tf.nn.relu(max_pool_2x2(h_conv1))
```

使用了一个 3*3*1*32  的filter, 输入的图像是1通道的，进行第一次卷积运算之后，图像的shape变成了 (-1, 28, 28, 32), 然后经过max pooling 之后，图像的大小变成了 (-1, 14, 14, 32), 可以通过以下代码看一下图像的大小。

```python
sess = tf.Session()
init = tf.global_variables_initializer()
sess.run(init)
sess.run(h_pool1, {x: train_imgs[0:1]}).shape
```

可以看到结果是 (1, 14, 14, 32)

第二层卷积层

```python
conv2_w = weight_variable([3, 3, 32, 64])
conv2_b = bias_variable([64])
h_conv2 = tf.nn.relu(conv2d(h_pool1, conv2_w) + conv2_b)
# after the convolution, the shape is (-1, 14, 14, 64)
h_pool2 = tf.nn.relu(max_pool_2x2(h_conv2))
# after the max pooling, the shape is (-1, 7, 7, 64)
```

第二层卷积层使用了一个 3*3*32*64 的filter, 经过这次卷积运算之后，图像会变成64通道的，图像的shape 变成了 (-1, 14, 14, 64), 然后通过max pooling 之后，图像的大小变为 (-1, 7, 7, 64)

可以通过上一层相同的方式查看这一步运算后的中间结果。

第一层全连接层

```python
h_flatten = tf.reshape(h_pool2, (-1, 7*7*64))
fc1_w = weight_variable([7*7*64, 1024])
fc1_b = bias_variable([1024])
h_fc1 = tf.nn.relu(tf.matmul(h_flatten, fc1_w) + fc1_b)
```

在进入全连接层之前，由于一张图片的数据是7*7*64的张量，需要先将张量铺平为1*3136的向量，再经过一层全连接层后，图片的大小变为(-1, 1024)

在第一层全连接之后加入drop out 来减少过拟合，drop out 的比例由参数来控制

```python
keep_prop = tf.placeholder(tf.float32)
h_drop1 = tf.nn.dropout(h_fc1, keep_prop)
```

第二层全连接层

```python
fc2_w = weight_variable([1024, 10])
fc2_b = weight_variable([10])
y_conv = tf.matmul(h_drop1, fc2_w) + fc2_b
```

在经过第二层的全连接层之后，输出变为了(-1, 10)的向量，正好对应着进行onehot编码之后的label。

y_conv 就是我们创建得包含两层卷积层，两层全连接层的神经网络结构。

### 训练

模型的驯良就是优化模型参数的过程，通过将模型的输出值(y_conv)与实际的label进行对比，不断的调整参数(卷积层和全连接层的w,b)来使得模型的输出与实际的label之间的误差(损失函数)逐渐降低的过程。梯度下降则是最常见的一种优化算法。

```python
y_ = tf.placeholder(tf.float32, [None, 10])
cross_entropy = tf.nn.softmax_cross_entropy_with_logits(labels=y_, logits=y_conv)
train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
```

上面一段代码直观的翻译过来就是，给label建立一个占位符，每一张图片的label是一个10维的向量。使用交叉熵作为目标函数，使用 学习率为1e-4 的 adam 算法来最小化交叉熵。这就是我们的训练过程。

可以通过以下方式使用100个样本对模型进行优化

```python
sess = tf.Session()
init = tf.global_variables_initializer()
sess.run(init)
sess.run(train_step, {x: train_imgs[0:100], keep_prop: 0.5, y_: train_labels[0:100]})
```

同时，为了观察训练都效果，我们需要评估模型的准确性，这儿我们使用acc来评价

```python
correct_prediction = tf.equal(tf.arg_max(y_conv, 1), tf.arg_max(y_, 1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
```

开始训练

```python
sess = tf.Session()
init = tf.global_variables_initializer()
sess.run(init)
train_round = 10000
for i in range(train_round):
    batch = mnist.train.next_batch(100)
    if i % 1000 == 0:
        acc = sess.run(accuracy, {x: batch[0], y_: batch[1], keep_prop: 0.5})
        print("step: %d, training accuracy %g" %(i, acc))
    sess.run(train_step, {x: batch[0], keep_prop: 0.5, y_: batch[1]})
acc = sess.run(accuracy, {x: mnist.test.images, y_: mnist.test.labels, keep_prop:1.0})
print("test accuracy %g" %acc)
```


