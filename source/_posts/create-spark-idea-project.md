title: 在IDEA中使用maven建立spark工程
date: 2017-05-02 22:30:14
categories: Spark
tags: [spark, idea, scala]
---

记录一下过程, 使用的软件是

IDEA 2016.3.4
Spark 1.6
maven 3.3.9
scala 2.10
JDK 1.8

### 使用maven新建工程

- File->New->Project
- 在弹出的页面中选择 Maven
- Project JDK 选择 1.8
- 勾选 Create from archetype
- 选择 net.alchim31.maven:scala-achetype-simple
- Next
- 填写 GroupId: com.xiangruix.test
- 填写 ArtifactId: sparksample
- Next
- Next
- Finish
- 等待项目的构建

### 修改 pom.xml 使得能编译通过

构建完成后需要修改pom.xml文件中的一些内容，
由于 Spark1.6 使用的是 Scala 2.10, 因此修改 properties 中 scala.version 为 2.10.5

```xml
<scala.version>2.10.5</scala.version>
```

新建的项目如果是使用的是net.alchim31.maven:scala-achetype-simple 1.4版本，那么测试框架的依赖会有点问题，需要将以下依赖进行修改, 如果net.alchim31.maven:scala-achetype-simple的版本选择的是1.6则在测试框架的依赖这是没问题的，就不需要改动,直接将scala版本改为2.10就行，

```xml
    <dependency>
      <groupId>org.specs2</groupId>
      <artifactId>specs2_${scala.version}</artifactId>
      <version>1.12.3</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.scalatest</groupId>
      <artifactId>scalatest_${scala.version}</artifactId>
      <version>2.0.M6-SNAP3</version>
      <scope>test</scope>
    </dependency>
```

修改为

```xml
    <dependency>
      <groupId>org.specs2</groupId>
      <artifactId>specs2-core_${scala.compat.version}</artifactId>
      <version>2.4.16</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.specs2</groupId>
      <artifactId>specs2-junit_${scala.compat.version}</artifactId>
      <version>2.4.16</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.scalatest</groupId>
      <artifactId>scalatest_${scala.compat.version}</artifactId>
      <version>2.2.4</version>
      <scope>test</scope>
    </dependency>
```

同时候 properties中增加

```xml
<scala.compat.version>2.10</scala.compat.version>
```

对于 scala 2.11 需要去掉 pom.xml 文件中的以下选项

```xml
<arg>-make:transitive</arg>
```

clean以下之后再compile发现能编译成功了

### 加入 spark 相关的依赖

```xml
    <dependency>
      <groupId>org.scala-lang</groupId>
      <artifactId>scala-reflect</artifactId>
      <version>${scala.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-core_${scala.compat.version}</artifactId>
      <version>${spark.version}</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-graphx_${scala.compat.version}</artifactId>
      <version>${spark.version}</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-mllib_${scala.compat.version}</artifactId>
      <version>${spark.version}</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-sql_${scala.compat.version}</artifactId>
      <version>${spark.version}</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-hive_${scala.compat.version}</artifactId>
      <version>${spark.version}</version>
      <scope>provided</scope>
    </dependency>
```

同时在 properties 中加入 spark 版本的值

```xml
<spark.version>1.6.2</spark.version>
```

现在 依赖添加完成，在App.scala中写几行代码测试以下

```scala
package com.xiangruix.test

import org.apache.log4j.{Level, Logger}
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.sql.hive.HiveContext

/**
 * @author ${user.name}
 */
object App {
  
  def main(args : Array[String]) {
    val sparkConf = new SparkConf().setAppName("spark test")
    val sc = new SparkContext(sparkConf)
    val hc = new HiveContext(sc)
    val log = Logger.getLogger(this.getClass)
    log.setLevel(Level.INFO)
    import hc.implicits._
    val rdd = sc.parallelize(Array(("user2", 22), ("user3", 33)))
    rdd.toDF("name", "age").show()
    log.info("hello, word!!!!")
  }
}
```

打包之后使用 spark-submit 提交 进行测试

```shell
spark-submit --class package com.xiangruix.test.App sparksample-1.0-SNAPSHOT.jar
```

运行结果

    +-----+---+
    | name|age|
    +-----+---+
    |user2| 22|
    |user3| 33|
    +-----+---+
    17/04/02 23:05:00 INFO App$: hello, word!!!!

### 总结

1. 在使用 net.alchim31.maven:scala-achetype-simple 创建的scala项目会有一点问题，添加的测试依赖需要进行一点改动才能编译通过

2. scala 和 spark 的版本需要匹配， spark1.6 使用的是 scala 2.10, 如果使用 spark 2.0 则需使用 scala 2.11
