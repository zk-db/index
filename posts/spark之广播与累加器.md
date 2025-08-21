---
title: spark之广播与累加器
date: 2022-10-25 15:30:43
tags:
  - spark
  - broadcast
  - accumulator
---



# 广播

如果我们要在分布式计算里面分发大对象，例如：字典，集合，黑白名单等，这个都会由Driver端进行分发，一般来讲，如果这个变量不是广播变量，那么每个task就会分发一份，这在**task数目十分多的情况下Driver的带宽会成为系统的瓶颈，而且会大量消耗task服务器上的资源**，如果将这个变量声明为广播变量，那么知识每个executor拥有一份，这个executor启动的task会共享这个变量，节省了通信的成本和服务器的资源。

## 使用方式

```scala
# create
val a = 3
val broadcast = sc.broadcast(a)

# use
val c = broadcast.value
```



## 注意事项

1、能不能将一个RDD使用广播变量广播出去？

    不能，因为RDD是不存储数据的。**可以将RDD的结果广播出去。**

2、 广播变量只能在Driver端定义，**不能在Executor端定义。**

3、 在Driver端可以修改广播变量的值，**在Executor端无法修改广播变量的值。**

4、如果executor端用到了Driver的变量，如果**不使用广播变量在Executor有多少task就有多少Driver端的变量副本。**

5、如果Executor端用到了Driver的变量，如果**使用广播变量在每个Executor中只有一份Driver端的变量副本。**



# 累加器

在spark应用程序中，我们经常会有这样的需求，如异常监控，调试，记录符合某特性的数据的数目，这种需求都需要用到计数器，如果一个变量不被声明为一个累加器，那么它将在被改变时不会再driver端进行全局汇总，即在分布式运行时每个task运行的只是原始变量的一个副本，并不能改变原始变量的值，但是当这个变量被声明为累加器后，该变量就会有分布式计数的功能。

## 使用方式

```scala
# create
val a = sc.accumulator(0)

# use
val b = a.value
```

## 注意事项

累加器在Driver端定义赋初始值，累加器只能在Driver端读取最后的值，在Excutor端更新。
