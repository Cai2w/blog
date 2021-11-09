---
title: JVM学习笔记(一)
date: 2021-10-03 00:06:20
tags:
- JVM
- 内存结构
categories:
- [Java,JVM]
description: 该篇文章包含内存结构的相关知识，其中包含程序计数器、虚拟机栈、本地方法栈、堆、方法区、直接内存
sticky:
cover: https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/undraw_developer_activity_bv83.png
---

## 引言

### 什么是JVM？

#### 定义

`Java Virtual Machine`，Java 程序的**运行环境**（Java 二进制字节码的运行环境）

#### 好处

- 一次编译，处处执行
- 自动的内存管理，垃圾回收机制
- 数组下标越界检查
- 多态：JVM内部使用虚方法调用的机制实现了多态

#### 比较

JVM、JRE、JDK 的关系如下图所示

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003172846.png)

### 学习JVM有什么用？

- 面试必备
- 中高级程序员必备
- 想走的长远，就需要懂原理，比如：自动装箱、自动拆箱是怎么实现的，反射是怎么实现的，垃圾回收机制是怎么回事，动态代理是怎么实现的......  JVM 是必须掌握的

### 常见的JVM

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003004321.png)

我们主要学习的是 HotSpot 版本的虚拟机。

### 学习路线

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003004423.png)

ClassLoader：Java 代码编译成二进制后，会经过类加载器，这样才能加载到 JVM 中运行。
Method Area：类是放在方法区中。
Heap：类的实例对象。
当类调用方法时，会用到 JVM Stacks、PC Register、本地方法栈。
方法执行时的每行代码是有执行引擎中的解释器逐行执行，方法中的热点代码频繁调用的方法，由 JIT 编译器优化后执行，GC 会对堆中不用的对象进行回收。需要和操作系统打交道就需要使用到本地方法接口。

## 内存结构

### 程序计数器

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003083245.png)

#### 定义

Program Counter Register 程序计数器（寄存器）

- 作用：是记录下一条 jvm 指令的执行地址行号
- 特点：
  - 是线程私有的，随着线程创建而创建，随着线程销毁而销毁
  - 不会存在内存溢出
  - 是一块较小的内存空间、

#### 作用

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003005633.png)

- 解释器会解释指令为机器码交给 cpu 执行，**程序计数器会记录下一条jvm指令的执行地址**，这样下一次解释器会从程序计数器拿到指令地址，获取对应指令然后进行解释执行。

> 程序计数器在物理上是通过寄存器来实现的，因为寄存器是整个cpu中读取速度最快的一个单元，我们读取指令地址这个动作是非常频繁的，所以java虚拟机在设计的时候把cpu中的寄存器当作程序计数器，用它来存储地址，将来来读取这个地址

### 虚拟机栈

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003083648.png)

#### 定义

Java Virtual Machine Stacks （Java 虚拟机栈）

- 每个线程运行需要的内存空间，称为虚拟机栈
- 每个栈由多个栈帧（Frame）组成，对应着每次调用方法时所占用的内存
- 每个线程只能有一个活动栈帧，对应着当前正在执行的方法

问题辨析：

1. 垃圾回收是否涉及栈内存？
   - 不会。栈内存是方法调用产生的，方法调用结束后会弹出栈 。
2. 栈内存分配越大越好吗？
   - 不是。因为物理内存是一定的，栈内存越大，可以支持更多的递归调用，但是可执行的线程数就会越少。
3. 方法内的局部变量是否线程安全
   1. 如果方法内部的变量没有逃离方法的作用访问，它是线程安全的
   2. 如果是局部变量引用了对象，并逃离了方法的访问，那就要考虑线程安全问题。

#### 栈内存溢出

- 栈帧过多导致栈内存溢出 

- 栈帧过大导致栈内存溢出

> 栈帧过大、过多、或者第三方类库操作，都有可能造成栈内存溢出 java.lang.stackOverflowError ，使用 `-Xss256k` 指定栈内存大小！一般都是栈帧过多导致内存溢出

#### 线程运行诊断

案例一：cpu 占用过多

解决方法：Linux 环境下运行某些程序的时候，可能导致 CPU 的占用过高，这时需要定位占用 CPU 过高的线程

- `top` 命令，查看是哪个进程占用 CPU 过高
- `ps H -eo pid, tid（线程id）, %cpu | grep 刚才通过 top 查到的进程号`（通过 ps 命令进一步查看是哪个线程占用 CPU 过高）
- `jstack 进程 id` 通过查看进程中的线程的 nid ，刚才通过 ps 命令看到的 tid 来对比定位，**注意 jstack 查找出的线程 id 是 16 进制的，需要转换。**

> 生产环境不推荐jstack，因为打印线程信息jvm会暂停其他线程

案例二：程序运行很长时间没有结果

可能是由于多个线程发生了死锁

我们同样可以使用jstack进行问题的定位

### 本地方法栈

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003091103.png)

> jvm调用一些本地方法时需要给这些本地方法提供的一个内存空间。一些带有 native 关键字的方法就是需要 JAVA 去调用本地的C或者C++方法，因为 JAVA 有时候没法直接和操作系统底层交互，所以需要用到本地方法栈，服务于带 native 关键字的方法。

### 堆

> - 程序计数器、虚拟机栈、本地方法栈都是线程私有的
> - 堆、方法区时线程共享的区

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003091601.png)

#### 定义

Heap 堆

- 通过new关键字创建的对象都会被放在堆内存

特点

- 它是线程共享，堆内存中的对象都需要考虑线程安全问题
- 有垃圾回收机制

#### 堆内存溢出

java.lang.OutofMemoryError ：java heap space. 堆内存溢出
可以使用 `-Xmx8m` 来指定堆内存大小。

#### 堆内存诊断

1. jps 工具
   - 查看当前系统中有哪些 java 进程
2. jmap 工具
   - 查看堆内存占用情况 `jmap -heap 进程id`
3. jconsole 工具
   - 图形界面的，多功能的监测工具，可以连续监测
4. jvisualvm 工具

### 方法区

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003100533.png)

#### 定义

Java 虚拟机有一个在所有 Java 虚拟机线程之间共享的方法区域。方法区域类似于用于传统语言的编译代码的存储区域，或者类似于操作系统进程中的“文本”段。它存储每个类的结构，例如运行时常量池、字段和方法数据，以及方法和构造函数的代码，包括特殊方法，用于类和实例初始化以及接口初始化方法区域是在虚拟机启动时创建的。尽管**方法区域在逻辑上是堆的一部分**，但简单的实现可能不会选择垃圾收集或压缩它。此规范不强制指定方法区的位置或用于管理已编译代码的策略。方法区域可以具有固定的大小，或者可以根据计算的需要进行扩展，并且如果不需要更大的方法区域，则可以收缩。方法区域的内存不需要是连续的！

> 官方解释：[JVM规范-方法区定义](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html)
>
> Oracle的HotSpot虚拟机在jdk1.8之前的实现叫做永久代，永久代就是使用堆内存的一部分作为方法区；但是在jdk1.8之后把永久代移除了，换了个实现，这个实现叫原空间，原空间用的不是堆的内存，用的是本地内存，也就是操作系统的内存
>
> **方法区是规范，永久代和原空间是实现**

#### 组成

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003113021.png)

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003113034.png)

#### 方法区内存溢出

- 1.8 之前会导致永久代内存溢出
  - 使用 `-XX:MaxPermSize=8m` 指定永久代内存大小
  - 演示永久代内存溢出java.lang.OutOfMemoryError: PermGen space
- 1.8 之后会导致元空间内存溢出
  - 使用 `-XX:MaxMetaspaceSize=8m` 指定元空间大小
  - 演示元空间内存溢出 java.lang.OutOfMemoryError: Metaspace

场景：

- spring
- mybatis
- ......

#### 运行时常量池

二进制字节码包含（类的基本信息，常量池，类方法定义，包含了虚拟机的指令）
首先看看常量池是什么，编译如下代码(`javac Test.java`)：

```java
public class Test {

    public static void main(String[] args) {
        System.out.println("Hello World!");
    }

}
```

然后使用 `javap -v Test.class` 命令反编译查看结果。

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003121254.png)

每条指令都会对应常量池表中一个地址，常量池表中的地址可能对应着一个类名、方法名、参数类型等信息。

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003121312.png)

**常量池**：
就是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量信息
**运行时常量池**：
常量池是 *.class 文件中的，当该类被加载以后，它的常量池信息就会放入运行时常量池，并把里面的符号地址变为真实地址

#### StringTable

先看几道面试题：

```java
String s1 = "a";
String s2 = "b";
String s3 = "ab";
String s4 = s1 + s2;
String s5 = "a" + "b";
String s6 = s4.intern();
// 问
System.out.println(s3 == s4); //false
System.out.println(s3 == s5);//true
System.out.println(s3 == s6);//true
String x2 = new String("c") + new String("d");
String x1 = "cd";
x2.intern();
// 问，如果调换了【最后两行代码】的位置呢，如果是jdk1.6呢
System.out.println(x1 == x2);//false
//如果调换最后两行代码的位置就是true
//如果是jdk1.6，调换两行代码的位置，x2.intern();会产生一个"cd"副本，将副本入常量池，而x2还是指向的堆中的new string("ab"),而x1指向的是常量池中的"cd",所以为false
```

反编译之后的分析：

```java
// StringTable [ "a", "b" ,"ab" ]  hashtable 结构，不能扩容
// 常量池中的信息，都会被加载到运行时常量池中， 这时 a b ab 都是常量池中的符号，还没有变为 java 字符串对象
// ldc #2 会把 a 符号变为 "a" 字符串对象
// ldc #3 会把 b 符号变为 "b" 字符串对象
// ldc #4 会把 ab 符号变为 "ab" 字符串对象

public static void main(String[] args) {
    String s1 = "a"; // 懒惰的
    String s2 = "b";
    String s3 = "ab";
    String s4 = s1 + s2; // new StringBuilder().append("a").append("b").toString()  new String("ab")
    String s5 = "a" + "b";  // javac 在编译期间的优化，结果已经在编译期确定为ab
    System.out.println(s3 == s5);//true
}
```

```java
//  ["ab", "a", "b"]
public static void main(String[] args) {

    String x = "ab";
    String s = new String("a") + new String("b");

    // 堆  new String("a")   new String("b") new String("ab")
    String s2 = s.intern(); // 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池， 会把串池中的对象返回
    System.out.println( s2 == x);//true
    System.out.println( s == x );//false
}
```

```java
//  ["a", "b", "ab"]
public static void main(String[] args) {

    String s = new String("a") + new String("b");

    // 堆  new String("a")   new String("b") new String("ab")
    String s2 = s.intern(); // 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池， 会把串池中的对象返回
    System.out.println( s2 == "ab");//true
    System.out.println( s == "ab" );//true
}
```

- 常量池中的字符串仅是符号，只有在被用到时才会转化为对象
- 利用串池的机制，来避免重复创建字符串对象
- 字符串变量拼接的原理是StringBuilder
- 字符串常量拼接的原理是编译器优化
- 可以使用intern方法，主动将串池中还没有的字符串对象放入串池中
  - 1.8 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池， 会把串池中的对象返回
  - 1.6 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有会把此对象复制一份，放入串池， 会把串池中的对象返回

> 无论放入是否成功，都会返回串池中的字符串对象

#### StringTable的位置

jdk1.6 StringTable 位置是在永久代中，1.8 StringTable 位置是在堆中。

> 永久代的回收效率低，只有在`Full GC`的时候才会触发垃圾回收，而`Full GC`要等到老年代的空间不足时才会触发

#### StringTable 垃圾回收

-Xmx10m 指定堆内存大小
-XX:+PrintStringTableStatistics 打印字符串常量池信息
-XX:+PrintGCDetails
-verbose:gc 打印 gc 的次数，耗费时间等信息

```java
/**
 * 演示 StringTable 垃圾回收
 * -Xmx10m -XX:+PrintStringTableStatistics -XX:+PrintGCDetails -verbose:gc
 */
public class Code_05_StringTableTest {

    public static void main(String[] args) {
        int i = 0;
        try {
            for(int j = 0; j < 10000; j++) { // j = 100, j = 10000
                String.valueOf(j).intern();
                i++;
            }
        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            System.out.println(i);
        }
    }

}

```

#### StringTable 性能调优

- 因为StringTable是由HashTable实现的，所以可以适当增加HashTable桶的个数，来减少字符串放入串池所需要的时间

> -XX:StringTableSize=桶个数（最少设置为 1009 以上）

- 考虑是否需要将字符串对象入池，可以通过 intern 方法减少重复入池

### 直接内存

#### 定义

直接内存（Direct Memory）就是系统内存

- 常见于 NIO 操作时，用于数据缓冲区
- 分配回收成本较高，但读写性能高
- 不受 JVM 内存回收管理

#### 使用直接内存的好处

文件读写流程：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003170018.png)

因为 java 不能直接操作文件管理，需要切换到内核态，使用本地方法进行操作，然后读取磁盘文件，会在系统内存中创建一个缓冲区，将数据读到系统缓冲区， 然后在将系统缓冲区数据，复制到 java 堆内存中。缺点是数据存储了两份，在系统内存中有一份，java 堆中有一份，造成了不必要的复制。

**使用了 DirectBuffer 文件读取流程**

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211003170052.png)

直接内存是操作系统和 Java 代码都可以访问的一块区域，无需将代码从系统内存复制到 Java 堆内存，从而提高了效率。

#### 直接内存回收原理

```java
public class Code_06_DirectMemoryTest {

    public static int _1GB = 1024 * 1024 * 1024;

    public static void main(String[] args) throws IOException, NoSuchFieldException, IllegalAccessException {
//        method();
        method1();
    }

    // 演示 直接内存 是被 unsafe 创建与回收
    private static void method1() throws IOException, NoSuchFieldException, IllegalAccessException {

        Field field = Unsafe.class.getDeclaredField("theUnsafe");
        field.setAccessible(true);
        Unsafe unsafe = (Unsafe)field.get(Unsafe.class);

        long base = unsafe.allocateMemory(_1GB);
        unsafe.setMemory(base,_1GB, (byte)0);
        System.in.read();

        unsafe.freeMemory(base);
        System.in.read();
    }

    // 演示 直接内存被 释放
    private static void method() throws IOException {
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_1GB);
        System.out.println("分配完毕");
        System.in.read();
        System.out.println("开始释放");
        byteBuffer = null;
        System.gc(); // 手动 gc
        System.in.read();
    }

}
```

直接内存的回收不是通过 JVM 的垃圾回收来释放的，而是通过`unsafe.freeMemory` 来手动释放。
第一步：allocateDirect 的实现

```java
public static ByteBuffer allocateDirect(int capacity) {
    return new DirectByteBuffer(capacity);
}
```

底层是创建了一个 DirectByteBuffer 对象。
第二步：DirectByteBuffer 类

```java
DirectByteBuffer(int cap) {   // package-private
   
    super(-1, 0, cap, cap);
    boolean pa = VM.isDirectMemoryPageAligned();
    int ps = Bits.pageSize();
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));
    Bits.reserveMemory(size, cap);

    long base = 0;
    try {
        base = unsafe.allocateMemory(size); // 申请内存
    } catch (OutOfMemoryError x) {
        Bits.unreserveMemory(size, cap);
        throw x;
    }
    unsafe.setMemory(base, size, (byte) 0);
    if (pa && (base % ps != 0)) {
        // Round up to page boundary
        address = base + ps - (base & (ps - 1));
    } else {
        address = base;
    }
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap)); // 通过虚引用，来实现直接内存的释放，this为虚引用的实际对象, 第二个参数是一个回调，实现了 runnable 接口，run 方法中通过 unsafe 释放内存。
    att = null;
}
```

这里调用了一个 Cleaner 的 create 方法，且后台线程还会对虚引用的对象监测，如果虚引用的实际对象（这里是 DirectByteBuffer ）被回收以后，就会调用 Cleaner 的 clean 方法，来清除直接内存中占用的内存。

```java
 public void clean() {
        if (remove(this)) {
            try {
            // 都用函数的 run 方法, 释放内存
                this.thunk.run();
            } catch (final Throwable var2) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        if (System.err != null) {
                            (new Error("Cleaner terminated abnormally", var2)).printStackTrace();
                        }

                        System.exit(1);
                        return null;
                    }
                });
            }

        }
    }
```

可以看到关键的一行代码， this.thunk.run()，thunk 是 Runnable 对象。run 方法就是回调 Deallocator 中的 run 方法

```java
		public void run() {
            if (address == 0) {
                // Paranoia
                return;
            }
            // 释放内存
            unsafe.freeMemory(address);
            address = 0;
            Bits.unreserveMemory(size, capacity);
        }
```

**直接内存的回收机制总结**

- 使用了 Unsafe 类来完成直接内存的分配回收，回收需要主动调用freeMemory 方法
- ByteBuffer 的实现内部使用了 Cleaner（虚引用）来检测 ByteBuffer 。一旦ByteBuffer 被垃圾回收，那么会由 ReferenceHandler（守护线程） 来调用 Cleaner 的 clean 方法调用 freeMemory 来释放内存

**注意：**

```java
/**
     * -XX:+DisableExplicitGC 显示的
     */
    private static void method() throws IOException {
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_1GB);
        System.out.println("分配完毕");
        System.in.read();
        System.out.println("开始释放");
        byteBuffer = null;
        System.gc(); // 手动 gc 失效
        System.in.read();
    }
```

一般用 jvm 调优时，会加上下面的参数：

```java
-XX:+DisableExplicitGC  // 静止显示的 GC
```

意思就是禁止我们手动的 GC，比如手动 System.gc() 无效，它是一种 full gc，会回收新生代、老年代，会造成程序执行的时间比较长。所以我们就通过 unsafe 对象调用 freeMemory 的方式释放内存

