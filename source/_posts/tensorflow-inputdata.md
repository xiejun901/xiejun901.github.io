---
title: tensorflow 数据读取
date: 2018-05-20 01:10:07
tags: [tensorflow, dataset, machine learning]
categories: machine learning
---




```python
import tensorflow as tf
import numpy as np
```

## tensorflow 读取数据

tensorflow 读取数据主要有以下几种方式:

- Feeding: 在运行的通过代码中的feed_dict来提供数据，这种方式在数据量小或者简单搭建一个网络调试的时候可以使用，但是如果数据两太大就比较麻烦了，需要自己去控制数据的读取，然后分batch输入到网络中

- QueueRunner: 基于队列的方式读取数据，可以一边使用数据一边从磁盘读取，现在已经完全可以使用tf.data 这一套api 来代替了

- tf.data : 这一套api提供了一个 Dataset的抽象，可以从文件，内存，已有的tensor里面读取数据生成dataset，然后在已有的dataset上面可以数据的解析，过滤，缓存，shuffle，控制batch的大小等操作，是最方便的一种读取数据的方法

### Dataset 的创建

dataset 可以认为是由很多行数据构成，每一行的结构是相同的，每一行包含了一个或者多个tensor。可以通过 Dataset.output_types 输出每一行的tensor的数据类型，Dataset.output_shapes输出每一行的tensor的shape。dataset的数据原可以从内存读入，也可以从磁盘读入。从磁盘读入的时候可以读取tfrecord格式的数据或者文本格式的数据。

这儿举个例子我们由一些数据，需要用来使用，这儿介绍以下几种方式从原始数据创建datasetd的方法

假设数据以文本方式存储，以空格进行分割，第一列是labe，第二列是一种feature,由以逗号分割的5个浮点数组成，第三列是另外一种feature，是以逗号分割的两个字符串构成

```
1 1.0,2.0,3.0,4.0,5.0 beijing,shanghai
0 6.0,7.0,8.0,9.0,10.0 chengdu,chongqing
```

#### 从内存中的数据创建

当我们的数据两很小的时候，我们可以直接把数据读入内存然后，创建dataset.



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

在有了tfrecord文件之后，我们就可以读取乘dataset了，读进来之后，dataset的每一个元素的类型是tf.string,因此是需要解析的


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

