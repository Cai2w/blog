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

```java
package com.caicai;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 * @author wangcaicai
 * @date 2021/11/18 0:51
 * @description 流处理
 */
public class StreamWordCount {
    public static void main(String[] args) throws Exception {
        //创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // 设置并行度，默认值 = 当前计算机的CPU逻辑核数（设置成1即单线程处理）
        // env.setMaxParallelism(4);

        //从文件中读取数据
        String inputPath = "E:\\java\\WorkSpace\\Flink\\src\\main\\resources\\hello.txt";
        DataStream<String> inputDataStream = env.readTextFile(inputPath);

        // 对数据集进行处理，按空格分词展开，转换成(word, 1)二元组进行统计
        // 按照第一个位置的word分组
        // 按照第二个位置上的数据求和
        DataStream<Tuple2<String, Integer>> resultStream = inputDataStream.flatMap(new MyFlatMapper())
                .keyBy(item -> item.f0)
                .sum(1);
        resultStream.print();

        //执行任务
        env.execute();
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

输出：

*这里因为是流处理，所以所有中间过程都会被输出，前面的序号就是并行执行任务的线程编号。*

```txt
9> (how,1)
1> (scala,1)
6> (storm,1)
6> (are,1)
4> (hello,1)
4> (hello,2)
2> (java,1)
4> (hello,3)
1> (spark,1)
7> (you,1)
10> (flink,1)
4> (hello,4)
4> (hello,5)
7> (fine,1)
7> (you,2)
5> (thank,1)
11> (and,1)
7> (you,3)
```

### 流式数据源测试

1. 通过`nc -lk <port>`打开一个socket服务，用于模拟实时的流数据

```shell
nc -lk 7777
```

2. 代码修改inputStream的部分

```java
package com.caicai;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 * @author wangcaicai
 * @date 2021/11/18 0:51
 * @description 流处理
 */
public class StreamWordCount {
    public static void main(String[] args) throws Exception {
        //创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // 设置并行度，默认值 = 当前计算机的CPU逻辑核数（设置成1即单线程处理）
        // env.setMaxParallelism(32);

        //从文件中读取数据
        // String inputPath = "E:\\java\\WorkSpace\\Flink\\src\\main\\resources\\hello.txt";
        // DataStream<String> inputDataStream = env.readTextFile(inputPath);

        // 从socket文本流读取数据
        DataStream<String> inputDataStream = env.socketTextStream("192.168.200.130", 7777);

        // 对数据集进行处理，按空格分词展开，转换成(word, 1)二元组进行统计
        // 按照第一个位置的word分组
        // 按照第二个位置上的数据求和
        DataStream<Tuple2<String, Integer>> resultStream = inputDataStream.flatMap(new MyFlatMapper())
                .keyBy(item -> item.f0)
                .sum(1);
        resultStream.print();

        //执行任务
        env.execute();
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

3. 在本地开启的socket中输入数据，观察IDEA的console输出。

   ![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211122162314.png)

    本人测试后发现，同一个字符串，前面输出的编号是一样的，因为key => hashcode,同一个key的hash值固定，分配给相对应的线程处理。

#### 优化修改

上面的代码，我们是把`host`和`port`写死在代码中的，这样其实不太好，我们可以设置在参数(`args`)中，借助`parameter tool`工具提取这些配置项

```java
//改动部分
import org.apache.flink.api.java.utils.ParameterTool;
// 用parameter tool工具从程序启动参数中提取配置项
ParameterTool parameterTool = ParameterTool.fromArgs(args);
String host = parameterTool.get("host");
int port = parameterTool.getInt("port");
// 从socket文本流读取数据
DataStream<String> inputDataStream = env.socketTextStream(host, port);
```

将配置项设置在`args`参数中

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211122163902.png)

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211122163930.png)

输入

```shell
--host 192.168.200.130 --port 7777
```

然后点击右下角的`apply`，再次运行程序就可以了

## Flink部署

### Standalone模式

> [Flink任务调度原理之TaskManager 与Slots](https://blog.csdn.net/qq_39657909/article/details/105823127) <= 下面内容出自该博文

1. Flink 中每一个 TaskManager 都是一个JVM进程，它可能会在独立的线程上执行一个或多个 subtask
2. 为了控制一个 TaskManager 能接收多少个 task， TaskManager 通过 task slot 来进行控制（一个 TaskManager 至少有一个 slot）
3. 每个task slot表示TaskManager拥有资源的一个固定大小的子集。假如一个TaskManager有三个slot，那么它会将其管理的内存分成三份给各个slot(注：这里不会涉及CPU的隔离，slot仅仅用来隔离task的受管理内存)
4. 可以通过调整task slot的数量去自定义subtask之间的隔离方式。如一个TaskManager一个slot时，那么每个task group运行在独立的JVM中。而**当一个TaskManager多个slot时，多个subtask可以共同享有一个JVM,而在同一个JVM进程中的task将共享TCP连接和心跳消息，也可能共享数据集和数据结构，从而减少每个task的负载**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428203404161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjU3OTA5,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428205219327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjU3OTA5,size_16,color_FFFFFF,t_70#pic_center)

1. 默认情况下，Flink 允许子任务共享 slot，即使它们是不同任务的子任务（前提是它们来自同一个job）。 这样的结果是，一个 slot 可以保存作业的整个管道。
2. Task Slot 是静态的概念，是指 TaskManager 具有的并发执行能力，可以通过参数taskmanager.numberOfTaskSlots进行配置；而并行度parallelism是动态概念，即TaskManager运行程序时实际使用的并发能力，可以通过参数parallelism.default进行配置。 举例：如果总共有3个TaskManager,每一个TaskManager中分配了3个TaskSlot,也就是每个TaskManager可以接收3个task,这样我们总共可以接收9个TaskSot。但是如果我们设置parallelism.default=1，那么当程序运行时9个TaskSlot将只有1个运行，8个都会处于空闲状态，所以要学会合理设置并行度！具体图解如下： ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200902165040619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjU3OTA5,size_16,color_FFFFFF,t_70#pic_center)

`conf/flink-conf.yaml`配置文件中

- `taskmanager.numberOfTaskSlots`
- `parallelism.default`

```yaml
# The number of task slots that each TaskManager offers. Each slot runs one parallel pipeline.

taskmanager.numberOfTaskSlots: 1

# The parallelism used for programs that did not specify and other parallelism.

parallelism.default: 1
```

注：**Flink存储State用的是堆外内存**，所以web UI里`JVM Heap Size`和`Flink Managed MEM`是两个分开的值。