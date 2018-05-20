---
title: tensorflow 数据读取
date: 2018-05-20 01:10:07
tags: [tensorflow, dataset, machine learning]
categories: machine learning
---


## tensorflow 读取数据

tensorflow 读取数据主要有以下几种方式:

- Feeding: 在运行的通过代码中的feed_dict来提供数据，这种方式在数据量小或者简单搭建一个网络调试的时候可以使用，但是如果数据两太大就比较麻烦了，需要自己去控制数据的读取，然后分batch输入到网络中

- QueueRunner: 基于队列的方式读取数据，可以一边使用数据一边从磁盘读取，现在已经完全可以使用tf.data 这一套api 来代替了

- tf.data : 这一套api提供了一个 Dataset的抽象，可以从文件，内存，已有的tensor里面读取数据生成dataset，然后在已有的dataset上面可以进行数据的解析，过滤，缓存，shuffle，控制batch的大小等操作，是最方便的一种读取数据的方法

### Dataset 的创建

dataset 可以认为是由很多行数据构成，每一行的结构是相同的，每一行包含了一个或者多个tensor。可以通过 Dataset.output_types 输出每一行的tensor的数据类型，Dataset.output_shapes输出每一行的tensor的shape。dataset的数据原可以从内存读入，也可以从磁盘读入。从磁盘读入的时候可以读取tfrecord格式的数据或者文本格式的数据。

这儿举个例子我们由一些数据，需要用来使用，这儿介绍以下几种方式从原始数据创建datasetd的方法

假设数据以文本方式存储，以空格进行分割，第一列是label，第二列是一种feature,由以逗号分割的5个浮点数组成，第三列是另外一种feature，是以逗号分割的两个字符串构成

```
1 1.0,2.0,3.0,4.0,5.0 beijing,shanghai
0 6.0,7.0,8.0,9.0,10.0 chengdu,chongqing
```

#### 从内存中的数据创建

当我们的数据两很小的时候，我们可以直接把数据读入内存然后，创建dataset.



```python
import tensorflow as tf
import numpy as np
sess = tf.Session()
```


```python
labels = []
features1 = []
features2 = []
with open("data.txt") as f:
    for line in f:
        splits = line.strip().split(" ")
        label = int(splits[0].strip())
        fe1 = list(map(lambda x: float(x.strip()), splits[1].strip().split(",")))
        fe2 = list(map(lambda x: x.strip(), splits[2].strip().split(",")))
        labels.append(label)
        features1.append(fe1)
        features2.append(fe2)                    
dataset = tf.data.Dataset.from_tensor_slices((labels, features1, features2))
print(dataset.output_types)
print(dataset.output_shapes)
```

    (tf.int32, tf.float32, tf.string)
    (TensorShape([]), TensorShape([Dimension(5)]), TensorShape([Dimension(2)]))


可以看到通过以上代码构造了一个dataset，dataset的每行的类型是 一个由 tf.int32, tf.float32, tf.string 构成的tuple. 同时也能看到dataset的shape

#### 从tfrecord格式的数据创建

也可以从存成tfrecord格式的文件中读取数据创建dataset, 这个地方由于没有存成tfrecord格式的数据，因此先生成一个。
tfrecord使用protobuffer来序列化和存储样本数据，定义了以下几个结构

```protobuf
message Feature{
    oneof kind{
        ByteList bytes_list = 1;
        FloatList float_list = 2;
        Int64List int64_list = 3;
    }
}
message Features{
    map<string, Feature> feature = 1;
}
message Example {
    Features features = 1;
}
message BytesList {
  repeated bytes value = 1;
}
message FloatList {
  repeated float value = 1 [packed = true];
}
message Int64List {
  repeated int64 value = 1 [packed = true];
}

```

可以将我们的数据用这几个结构来存储

通过以下代码生成 tfrecord 文件



```python
def _int64_feature(x):
    return tf.train.Feature(int64_list=tf.train.Int64List(value=x))
def _float_feature(x):
    return tf.train.Feature(float_list=tf.train.FloatList(value=x))
def _bytes_feature(x):
    return tf.train.Feature(bytes_list=tf.train.BytesList(value=x))
writer = tf.python_io.TFRecordWriter("data.tfrecords")
with open("data.txt") as f:
    for line in f:
        splits = line.strip().split(" ")
        label = int(splits[0].strip())
        fe1 = list(map(lambda x: float(x.strip()), splits[1].strip().split(",")))
        fe2 = list(map(lambda x: x.strip().encode(), splits[2].strip().split(",")))
        example = tf.train.Example(
            features=tf.train.Features(
                feature={
                    "label": _int64_feature([label]),
                    "feature1": _float_feature(fe1),
                    "feature2": _bytes_feature(fe2)
                }
            )
        )
        writer.write(example.SerializeToString())
writer.close()
```

在有了tfrecord文件之后，我们就可以读取数据创建dataset了，读进来之后，dataset的每一个元素的类型是tf.string,因此是需要解析的


```python
def _parse_function(example_proto):
    features={
        "label": tf.FixedLenFeature((), tf.int64),
        "feature1": tf.FixedLenFeature((5), tf.float32),
        "feature2": tf.FixedLenFeature((2), tf.string)
    }
    parsed_feature = tf.parse_single_example(example_proto,features)
    return parsed_feature["label"], parsed_feature["feature1"], parsed_feature["feature2"]  
dataset = tf.data.TFRecordDataset("data.tfrecords")
print(dataset.output_types)
print(dataset.output_shapes)
dataset = dataset.map(_parse_function)
print(dataset.output_types)
print(dataset.output_shapes)
```

    <dtype: 'string'>
    ()
    (tf.int64, tf.float32, tf.string)
    (TensorShape([]), TensorShape([Dimension(5)]), TensorShape([Dimension(2)]))


#### 从文本数据读入

dataset也可以直接从文本文件里面读取数据，进行解析。读进来每个元素是一个tf.string类型的tensor，然后可以对它进行解析等操作。不过由于tensorflow的字符串处理函数不是很好用，所有在这个地方可以通过tf.py_func使用python的函数来解析字符串



```python
def _parse_line_py(line):
    splits = line.split(b" ")
    label = int(splits[0].strip())
    fe1 = list(map(lambda x: float(x.strip()), splits[1].strip().split(b",")))
    fe2 = list(map(lambda x: x.strip(), splits[2].strip().split(b",")))
    return label, fe1, fe2

def _parse_function(line):
    return tf.py_func(_parse_line_py, [line], [tf.int64, tf.float64, tf.string])
dataset = tf.data.TextLineDataset(["data.txt"])
print(dataset.output_shapes)
print(dataset.output_types)
dataset = dataset.map(_parse_function)
print(dataset.output_shapes)
print(dataset.output_types)   
```

    ()
    <dtype: 'string'>
    (TensorShape(None), TensorShape(None), TensorShape(None))
    (tf.int64, tf.float64, tf.string)


### Dataset 的转换

在已经存在的dataset上，我们可以使用各种转换操作，将一个daset转换乘另外一个dataset，主要有以下这些操作

#### Dataset.map

使用map操作可以对dataset中的每个元素进行一个操作，变成另外一个dataset，例如上面的解析操作就是通过 map 来完成的。 map操作中有一个参数 num_parallel_calls 来控制并行度，通常我们可以将这个参数设置为cpu的核数，来最快速度的完成数据的预处理。如果这个地方并行度太低，有可能会导致cpu读取速度太慢，gpu在进行完运算之后需要等待数据，成为系统的瓶颈

#### Dataset.cache

cache操作，可以将dataset缓存在内存或者磁盘上，如果dataset的大小比较小或者是解析非常费事，可以使用这个来缓存结果。

#### Dataset.prefetch 

提前获取数据存在缓存里来减少gpu因为缺少数据而等待的情况。

#### Dataset.shuffle 

对数据进行打乱

#### Dataset.repeat

对数据进行重复，如果不使用repeat，在数据处理完一个epoch后会扔出tf.errors.OutOfRangeError的异常。如果同时有shuffle和repeat操作，将shuffle放在repeat前面可以有更好的数据顺序保证，将repeat放在shuffle前面会有更好的性能，官方文档上面建议将 shuffle放在前面

#### Dataset.batch 

对数据划分batch， 参数给n就表示一次提取出n条数据

一些关于dataset使用性能方面的说明可以看这里 [Input Pipeline Performance Guide](https://www.tensorflow.org/performance/datasets_performance)


### Dataset 的使用

构建好了dataset之后主要是通过Iterator来使用的。现在主要由以下几种使用方式

- one-shot
- initializable
- reinitializable
- feedable

在有了iterator之后，是通过 iterator.get_next()来使用迭代器中的值

#### one-shot

one-shot的方式是最简单的方式，通过dataset生成iterator，然后这个iterator只能从这个dataset中读取数据， 代码使用上面从text中读取的数据作为例子. 当我们读取完dataset中的所有数据的时候会抛出 tf.errors.OutOfRangeError 异常，我们可以通过捕获这个异常来控制epochs


```python
iterator = dataset.make_one_shot_iterator()
label, feature1, feature2 = iterator.get_next()
while True:
    try:
        print(sess.run([label, feature1, feature2]))
    except tf.errors.OutOfRangeError:
        print("End of dataset")
        break
```

    [1, array([ 1.,  2.,  3.,  4.,  5.]), array([b'beijing', b'shanghai'], dtype=object)]
    [0, array([  6.,   7.,   8.,   9.,  10.]), array([b'chengdu', b'chongqing'], dtype=object)]
    [1, array([ 1.,  2.,  3.,  4.,  5.]), array([b'beijing', b'shanghai'], dtype=object)]
    End of dataset


#### initializable

通过这种方式使用iterator需要显式的去调用 iterator.initializer来进行初始化，不过由于需要自己初始化，也就提供了在初始化的时候使用不同的参数来定义dataset的能力,例如我们可以通过不同的文件名来构建dataset



```python
filenames = tf.placeholder(tf.string)
dataset = tf.data.TextLineDataset(filenames).map(_parse_function)
iterator = dataset.make_initializable_iterator()
label, feature1, feature2 = iterator.get_next()
sess.run(iterator.initializer, feed_dict={filenames: ["data.txt"]})
print(sess.run([label, feature1, feature2]))
```

    [1, array([ 1.,  2.,  3.,  4.,  5.]), array([b'beijing', b'shanghai'], dtype=object)]


#### reinitializable

initializable 提供了使用不同的参数来创建dataset的能力，reinitializable则提供了使用不同的dataset来初始化iterator的能力，这个要求使用的不同的dataset的output_shape 和 output_type是相同的。


```python
# 直接使用Dataset.range 构造两个简单的dataset, 当然也可以使用前面提到过的其他方式来构造
dataset1 = tf.data.Dataset.range(1, 5)
dataset2 = tf.data.Dataset.range(6, 15)
iterator = tf.data.Iterator.from_structure(
    dataset1.output_types, dataset1.output_shapes)
x = iterator.get_next()
# 使用第一个dataset初始化
init_op_1 = iterator.make_initializer(dataset1)
sess.run(init_op_1)
print(sess.run(x))
# 使用第二个dataset初始化
init_op_2 = iterator.make_initializer(dataset2)
sess.run(init_op_2)
print(sess.run(x))
```

    1
    6


#### feedable

feedable 提供的是在sess.run()的时候选择哪个iterator的能力。



```python
# 创建要使用的数据和数据对应的iterator
dataset1 = tf.data.Dataset.range(1, 5)
dataset2 = tf.data.Dataset.range(6, 15)
iterator1 = dataset1.make_one_shot_iterator()
iterator2 = dataset2.make_initializable_iterator() # initializable生成的iterator需要初始化
handle1 = sess.run(iterator1.string_handle())
handle2 = sess.run(iterator2.string_handle())
# 创建要使用的iterator, 通过 handle的方式留出placheholder供后边初始化
handle = tf.placeholder(tf.string)
iterator = tf.data.Iterator.from_string_handle(
    handle,dataset1.output_types, dataset1.output_shapes)
x = iterator.get_next()
# 使用handle1来初始化iterator并使用
print(sess.run(x, feed_dict={handle: handle1}))
# 使用handle2来初始化iterator并使用， iterator在使用前需要先初始化
sess.run(iterator2.initializer)
print(sess.run(x, feed_dict={handle: handle2}))
```

    1
    6


#### 总结一下

- one-shot:

直接使用dataset创建iterator，不需要初始化，但是iterator创建好后对应的dataset就不会由任何改变了

只能通过编写不同的代码来控制使用的数据

- initializable

使用dataset来创建iterator，iterator需要初始化之后才能使用，但是在创建dataset的时候可以使用参数，然后在初始化iterator的时候通过feed_dict的方式来提供参数控制dataset的创建。

通过使用不同参数来控制使用的数据

- reinitializable

创建好一个iterator之后，这个iterator可以使用不同的dataset来初始化

通过使用不同的dataset来控制使用的数据

- feedable

通过 handle，iterator的类型和shape来构造iterator，然后在使用的时候可以使用不同的iterator的handle来初始化。由于iterator是可以使用不同参数来初始化的，因此这种方式提供最大的灵活度。

通过使用不同的iterator来控制使用的数据，iterator通过string_handle来表示


