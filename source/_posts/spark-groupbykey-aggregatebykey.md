title: spark 中 groupByKey 和 aggregateByKey(reduceByKey) 差距的一次真实体验
date: 2017-11-05 22:08:36
tags: [Spark, RDD, groupByKey, aggregateByKey, reduceByKey]
---

在spark中，groupByKey 需要将同一个key的记录全部拿到一块然后放入内存中进行计算，aggregateByKey 是现在各个partition会按照seqOp 先进性合并，然后再按照combineOp对各个partitions上计算的结果再进行组合。
因此aggregateByKey 拥有更高的效率。

之前由于一般按aggregateByKey写起来比较简单，不比groupByKey的写法麻烦，通常就按aggregateByKey的写法写了，也没有体验到他们的性能差别，这次有一段代码由于用aggregateByKey写起来要稍微不直观一点，因此就偷懒按照groupByKey来写了，同时由于数据量特别大，直接导致任务失败了，才按照aggregateByKey去优化了一下，没想到效率大大的提高，算是切身体验到了两种做法的效率差距。

问题是这样的, 我有两份特征数据，都是对应到人和经纬度的，现在需要把他们进行合并，合并的方式是对数据A中的每条记录a找到数据B中同一个人的距离a距离在500米以内并且访问次数最多的那条记录b,然后将a,b合并之后作为特征。

最直观的代码写法是如下这样的

```scala
def merge1[A: ClassTag, B: ClassTag, K: ClassTag](rddA: RDD[A], rddB: RDD[B])(
  ka: A => K, kb: B => K)(
  filterOp: (A, B) => Boolean, selectOp: (B, B) => B) = {
  val groupedB = rddB
    .map(x => (kb(x), x))
    .groupByKey()
  rddA.map(x => (ka(x), x))
    .leftOuterJoin(groupedB)
    .map {
      case (k, (a, Some(bs))) =>
        bs.filter(filterOp(a, _)).toList match {
          case Nil => (a, None)
          case bs => (a, bs.reduce(selectOp))
        }
      case (k, (a, None)) => (a, None)
    }
}
```

使用reduceByKey的方式，reduceByKey和aggregateByKey类似，都不需要将所有的记录汇总在一起进行计算，因此效率上也是能有很大的的提高的。这个地方使用reduceByKey就可以完成功能了，因此就使用了reduceByKey, 改写成 aggregateByKey 也是完全没有问题的。

```scala
def merge2[A: ClassTag, B: ClassTag, K: ClassTag](rddA: RDD[A], rddB: RDD[B])(
  ka: A => K, kb: B => K)(
  filterOp: (A, B) => Boolean, selectOp: (B, B) => B) = {
  val rddB_ = rddB.map(x => (kb(x), x))
  rddA.map(x => (ka(x), x))
    .leftOuterJoin(rddB_)
    .map {
      case (k, (a, bOpt)) =>
        (a, (a, bOpt))
    }
    .reduceByKey {
      case ((a, Some(b1)), (_, Some(b2))) =>
        if(filterOp(a, b1) && filterOp(a, b2)) {
          (a, Some(selectOp(b1, b2)))
        } else if(filterOp(a, b1)) {
          (a, Some(b1))
        } else if(filterOp(a, b2)) {
          (a, Some(b2))
        } else {
          (a, None)
        }
      case ((a, None), (_, Some(b2))) =>
        if(filterOp(a, b2))  (a, Some(b2))
        else (a, None)
      case ((a, Some(b1)), (_, None)) =>
        if(filterOp(a, b1)) (a, Some(b1))
        else (a, None)
      case ((a, None), (_, None)) => (a, None)
    }.values
}
```

下面给个所有的源代码和测试的例子,跑了之后可以发现两种方法都能达到想要的效果，在我自己真实的任务中，代码稍微改动，但是主体逻辑是按照这个写的,数据A的大小压缩后大概是150G, 数据B的大小是125G, 合并之后的数据是762G, 这些数据在内存中大概会占到好几个T，算是比较大的数据了。两边使用相同的配置，方法1在执行的过程中一直报内存错误，并且最终执行2个小时4分钟之后失败了。方法2执行过程中有少量内存错误，在1个小时13分钟之后执行完成。可以看到还是有很大的区别的。

```
object App {
  
  def main(args : Array[String]) {
    val conf = new SparkConf().setAppName("spark-2.1.1 test")
        .setMaster("local[2]")
    val sc = new SparkContext(conf)
    type A = (Int, Int)
    type B = (Int, Int, Int)
    val as = List((1, 2), (1, 10), (2, 2), (3, 10))
    val bs = List((1, 3, 3), (1, 1, 5), (1, 15, 10))
    val rddA = sc.parallelize(as)
    val rddB = sc.parallelize(bs)
    val  filterOp = (a:A, b:B) => {
      math.abs(a._2 - b._2) <= 2
    }
    val selectOp = (b1:B, b2:B) => if(b1._3 > b2._3) b1 else b2
    val resultA = merge1(rddA, rddB)(_._1, _._1)(filterOp, selectOp)
    val resultB = merge2(rddA, rddB)(_._1, _._1)(filterOp, selectOp)
    resultA.collect().foreach(println)
    resultB.collect().foreach(println)
  }



  def merge1[A: ClassTag, B: ClassTag, K: ClassTag](rddA: RDD[A], rddB: RDD[B])(
    ka: A => K, kb: B => K)(
    filterOp: (A, B) => Boolean, selectOp: (B, B) => B) = {
    val groupedB = rddB
      .map(x => (kb(x), x))
      .groupByKey()
    rddA.map(x => (ka(x), x))
      .leftOuterJoin(groupedB)
      .map {
        case (k, (a, Some(bs))) =>
          bs.filter(filterOp(a, _)).toList match {
            case Nil => (a, None)
            case bs => (a, Some(bs.reduce(selectOp)))
          }
        case (k, (a, None)) => (a, None)
      }
  }

  def merge2[A: ClassTag, B: ClassTag, K: ClassTag](rddA: RDD[A], rddB: RDD[B])(
    ka: A => K, kb: B => K)(
    filterOp: (A, B) => Boolean, selectOp: (B, B) => B) = {
    val rddB_ = rddB.map(x => (kb(x), x))
    rddA.map(x => (ka(x), x))
      .leftOuterJoin(rddB_)
      .map {
        case (k, (a, bOpt)) =>
          (a, (a, bOpt))
      }
      .reduceByKey {
        case ((a, Some(b1)), (_, Some(b2))) =>
          if(filterOp(a, b1) && filterOp(a, b2)) {
            (a, Some(selectOp(b1, b2)))
          } else if(filterOp(a, b1)) {
            (a, Some(b1))
          } else if(filterOp(a, b2)) {
            (a, Some(b2))
          } else {
            (a, None)
          }
        case ((a, None), (_, Some(b2))) =>
          if(filterOp(a, b2))  (a, Some(b2))
          else (a, None)
        case ((a, Some(b1)), (_, None)) =>
          if(filterOp(a, b1)) (a, Some(b1))
          else (a, None)
        case ((a, None), (_, None)) => (a, None)
      }.values
  }
}
```