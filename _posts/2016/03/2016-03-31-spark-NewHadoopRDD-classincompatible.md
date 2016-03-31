---
layout: post
title:  "spark API NewHadoopRDD 出现 java.io.InvalidClassException， local class incompatible 异常"
date:   '2016-03-31 14:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'spark'
excerpt: 'spark API NewHadoopRDD 出现 local class incompatible 异常'
keywords: 'spark NewHadoopRDD java.io.InvalidClassException  local-class-incompatible'
---

### 概述

近日在编写一个spark批量操作hbase库的时候，使用client 提交standalone模式的集群的时候，出现了如下错误：

```
WARN TaskSetManager: Lost task 1.0 in stage 0.0 (TID 1, 10.16.255.211): java.io.InvalidClassException: org.apache.spark.rdd.NewHadoopRDD; local class incompatible: stream classdesc serialVersionUID = 6559201105987478974, local class serialVersionUID = -8749574334148360781
```

<!--more-->

### 详细错误信息

```
WARN TaskSetManager: Lost task 1.0 in stage 0.0 (TID 1, 10.16.255.211): java.io.InvalidClassException: org.apache.spark.rdd.NewHadoopRDD; local class incompatible: stream classdesc serialVersionUID = 6559201105987478974, local class serialVersionUID = -8749574334148360781
at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:617)
at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1622)
at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1517)
at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1771)
at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1350)
at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:1990)
at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:1915)
at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1798)
at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1350)
at java.io.ObjectInputStream.readObject(ObjectInputStream.java:370)
at scala.collection.immutable.$colon$colon.readObject(List.scala:362)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:606)
at java.io.ObjectStreamClass.invokeReadObject(ObjectStreamClass.java:1017)
at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:1893)
at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1798)
at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1350)
at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:1990)
at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:1915)
at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1798)
at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1350)
at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:1990)
at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:1915)
at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1798)
at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1350)
at java.io.ObjectInputStream.readObject(ObjectInputStream.java:370)
at org.apache.spark.serializer.JavaDeserializationStream.readObject(JavaSerializer.scala:72)
at org.apache.spark.serializer.JavaSerializerInstance.deserialize(JavaSerializer.scala:98)
at org.apache.spark.scheduler.ResultTask.runTask(ResultTask.scala:61)
at org.apache.spark.scheduler.Task.run(Task.scala:88)
at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:214)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
at java.lang.Thread.run(Thread.java:745)
```

### 集群环境
运行环境、集群环境、程序依赖包具体信息如下：

```
os： centos 6.7
scala version 2.10.5
sbt version 0.13.8

集群环境：
cdh 5.5.1
集群 spark version 1.5.0
集群 hbase version 1.0.0
集群 hadoop version 2.6.0

程序依赖包中的version：
spark version 1.5.0   //原生的lib包
hbase version 1.0.0-cdh5.5.1
hadoop version 2.6.0-cdh5.5.1
```

### 问题产生的原因和解决方案

异常中给出的信息可以明显地看出，这个错误是因为本地的lib中的NewHadoopRDD类跟集群的不在同一个版本上的。
但奇怪的是明明集群的spark版本跟本地使用的spark库的版本是一致的为何还出现serialVersionUID不一致呢？

想到可能因为cdh做过二次开发或者打上了补丁等，我尝试着修改我本地的spark lib库的版本。
我先是尝试了使用集群上spark 的lib库，但发现导入编译后出现一堆报错，更别说运行了，所以否定了这个解决方案。

这个时候只有使用spark 原生的lib了，但我不太抱有希望。因为spark 1.4跟1.5可能有较大的改变，我没有使用1.4的，而是使用了原生spark 1.5.1 的lib库。
在导入编译后发现没有报错。提交运行，连接master没有问题，这说明1.5.0跟1.5.1在rpc上是一样的，或者改变不大。而在资源分配部分很快就过去了（原先卡在这里后就报错了，可能分配资源后进行了代码的反序列化和对比操作），程序运行通过。成功读取了hbase的数据。

上述说明，可能cdh 的spark可能是打过补丁的，因为cdh的版本5.5.1，spark为1.5.0，如果没有改变，使用spark 1.5.0的库应该是可以通过的。

### 总结

如果遇到这个问题：

```
java.io.InvalidClassException: xxx.xxx.xxx; local class incompatible: stream classdesc serialVersionUID = 6559201105987478974, local class serialVersionUID = -8749574334148360781
```

那么是库的版本不一致做成的，检查是哪个库出现的问题和你使用的这个库的版本是否跟集群上的对应。
