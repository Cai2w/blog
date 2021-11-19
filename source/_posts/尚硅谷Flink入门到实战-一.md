---
title: 尚硅谷Flink入门到实战(一)
date: 2021-11-18 00:18:27
tags:
- Flink
categories:
- Flink
description:
sticky:
cover: https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/undraw_good_team_m7uu.png
---

> [尚硅谷2021最新Java版Flink](https://www.bilibili.com/video/BV1qy4y1q728)
>
> 下面笔记来源（尚硅谷公开资料、网络博客、个人小结）
>
> 中间会把自己认为较重要的点做做标记（下划线、加粗等）

## Flink的特点

- 事件驱动（Event-driven）

- 基于流处理

  一切皆由流组成，离线数据是有界的流；实时数据是一个没有界限的流。（有界流、无界流）

- 分层API

  - 越顶层越抽象，表达含义越简明，使用越方便
  - 越底层越具体，表达能力越丰富，使用越灵活

### Flink vs Spark Streaming

- 数据模型
  - Spark采用RDD模型，spark streaming的DStream实际上也就是一组组小批数据RDD的集合
  - flink基本数据模型是数据流，以及事件（Event）序列
- 运行时架构
  - spark是批计算，将DAG划分为不同的stage，一个完成后才可以计算下一个
  - flink是标准的流执行模式，一个事件在一个节点处理完后可以直接发往下一个节点处理

## 快速上手

### 批处理实现WordCount

> flink-streaming-java_2.12:1.12.1 => org.apache.flink:flink-runtime_2.12:1.12.1 => com.typesafe.akka:akka-actor_2.12:2.5.21，**akka就是用scala实现的。即使这里我们用java语言，还是用到了scala实现的包**

**pom依赖**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.caicai</groupId>
    <artifactId>Flink</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <flink.version>1.12.0</flink.version>
        <scala.binary.version>2.14</scala.binary.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
    </dependencies>

</project>
```

**准备工作**

> 首先准备一个文件，存放一些简单的数据，以便后续Flink计算分析。在`resources`目录下新建一个`hello.txt`文件，并存入一些数据

```txt
hello java
hello flink
hello scala
hello spark
hello storm
how are you
fine thank you
and you
```

**代码实现**

```java
package com.caicai;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.operators.Order;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

/**
 * @author wangcaicai
 * @date 2021/11/18 0:51
 * @description 批处理
 */
public class WordCount {
    public static void main(String[] args) throws Exception {
        //创建执行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //从文件中读取数据
        String inputPath = "E:\\java\\WorkSpace\\Flink\\src\\main\\resources\\hello.txt";
        DataSet<String> inputDataSet = env.readTextFile(inputPath);

        // 对数据集进行处理，按空格分词展开，转换成(word, 1)二元组进行统计
        // 按照第一个位置的word分组
        // 按照第二个位置上的数据求和
        DataSet<Tuple2<String, Integer>> resultSet = inputDataSet.flatMap(new MyFlatMapper())
                .groupBy(0)
                .sum(1);
        resultSet.print();
    }

    // 自定义类，实现FlatMapFunction接口
    public static class MyFlatMapper implements FlatMapFunction<String, Tuple2<String, Integer>> {

        @Override
        public void flatMap(String s, Collector<Tuple2<String, Integer>> out) throws Exception {
            // 按空格分词
            String[] words = s.split(" ");
            // 遍历所有word，包成二元组输出
            for (String str : words) {
                out.collect(new Tuple2<>(str, 1));
            }
        }
    }
}
```

**输出**

```txt
(thank,1)
(spark,1)
(and,1)
(java,1)
(storm,1)
(flink,1)
(fine,1)
(you,3)
(scala,1)
(are,1)
(how,1)
(hello,5)
```

> [解决 Flink 升级1.12 报错 No ExecutorFactory found to execute the application](https://blog.csdn.net/qq_41398614/article/details/107553604)

### 流处理实现

在2.1批处理的基础上，新建一个类进行改动。

- 批处理=>几组或所有数据到达后才处理；流处理=>有数据来就直接处理，不等数据堆叠到一定数量级
- **这里不像批处理有groupBy => 所有数据统一处理，而是用流处理的keyBy => 每一个数据都对key进行hash计算，进行类似分区的操作，来一个数据就处理一次，所有中间过程都有输出！**
- **并行度：开发环境的并行度默认就是计算机的CPU逻辑核数**

代码实现

```

```

