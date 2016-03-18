---
layout: post
title:  "intellij远程提交任务到spark集群"
date:   '2016-03-16 14:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'intellij'
excerpt: 'intellij远程提交任务到spark集群'
keywords: 'intellij 远程提交任务 spark集群'
---

### 概述

本文介绍如何使用 intellij idea 远程提交任务到配置standnode 的spark集群上运行。
当你的程序抛出以下异常的时候，或者本文是你要找的解决方案了。
“WARN TaskSetManager: Lost task 1.0 in stage 0.0 (TID 1, 10.200.2.213): java.lang.ClassNotFoundException: top.spoofer.hbrdd.HbMain$$anonfun$1”

<!--more-->

### 提交任务抛出异常

异常如下：

```
16/03/17 14:29:10 INFO BlockManagerInfo: Added broadcast_1_piece0 in memory on 10.200.2.213:52531 (size: 1868.0 B, free: 530.3 MB)
16/03/17 14:29:10 INFO BlockManagerInfo: Added broadcast_1_piece0 in memory on 10.200.2.211:58199 (size: 1868.0 B, free: 530.3 MB)
16/03/17 14:29:10 WARN TaskSetManager: Lost task 1.0 in stage 0.0 (TID 1, 10.200.2.213): java.lang.ClassNotFoundException: top.spoofer.hbrdd.HbMain$$anonfun$1
	at java.net.URLClassLoader$1.run(URLClassLoader.java:366)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:270)
	at org.apache.spark.serializer.JavaDeserializationStream$$anon$1.resolveClass(JavaSerializer.scala:67)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1612)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1517)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1771)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1350)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:1990)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:1915)
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

16/03/17 14:29:10 INFO TaskSetManager: Starting task 1.1 in stage 0.0 (TID 2, 10.200.2.212, ANY, 2152 bytes)
16/03/17 14:29:10 INFO TaskSetManager: Lost task 0.0 in stage 0.0 (TID 0) on executor 10.200.2.211: java.lang.ClassNotFoundException (top.spoofer.hbrdd.HbMain$$anonfun$1) [duplicate 1]
16/03/17 14:29:10 INFO TaskSetManager: Starting task 0.1 in stage 0.0 (TID 3, 10.200.2.212, ANY, 2152 bytes)
16/03/17 14:29:10 INFO BlockManagerInfo: Added broadcast_1_piece0 in memory on 10.200.2.212:48171 (size: 1868.0 B, free: 530.3 MB)
16/03/17 14:29:11 INFO TaskSetManager: Lost task 1.1 in stage 0.0 (TID 2) on executor 10.200.2.212: java.lang.ClassNotFoundException (top.spoofer.hbrdd.HbMain$$anonfun$1) [duplicate 2]
16/03/17 14:29:11 INFO TaskSetManager: Starting task 1.2 in stage 0.0 (TID 4, 10.200.2.211, ANY, 2152 bytes)
16/03/17 14:29:11 INFO TaskSetManager: Lost task 0.1 in stage 0.0 (TID 3) on executor 10.200.2.212: java.lang.ClassNotFoundException (top.spoofer.hbrdd.HbMain$$anonfun$1) [duplicate 3]
16/03/17 14:29:11 INFO TaskSetManager: Starting task 0.2 in stage 0.0 (TID 5, 10.200.2.213, ANY, 2152 bytes)
16/03/17 14:29:11 INFO TaskSetManager: Lost task 1.2 in stage 0.0 (TID 4) on executor 10.200.2.211: java.lang.ClassNotFoundException (top.spoofer.hbrdd.HbMain$$anonfun$1) [duplicate 4]
16/03/17 14:29:11 INFO TaskSetManager: Starting task 1.3 in stage 0.0 (TID 6, 10.200.2.212, ANY, 2152 bytes)
16/03/17 14:29:11 INFO TaskSetManager: Lost task 0.2 in stage 0.0 (TID 5) on executor 10.200.2.213: java.lang.ClassNotFoundException (top.spoofer.hbrdd.HbMain$$anonfun$1) [duplicate 5]
16/03/17 14:29:11 INFO TaskSetManager: Starting task 0.3 in stage 0.0 (TID 7, 10.200.2.211, ANY, 2152 bytes)
16/03/17 14:29:11 INFO TaskSetManager: Lost task 1.3 in stage 0.0 (TID 6) on executor 10.200.2.212: java.lang.ClassNotFoundException (top.spoofer.hbrdd.HbMain$$anonfun$1) [duplicate 6]
16/03/17 14:29:11 ERROR TaskSetManager: Task 1 in stage 0.0 failed 4 times; aborting job
16/03/17 14:29:11 INFO TaskSchedulerImpl: Cancelling stage 0
16/03/17 14:29:11 INFO TaskSchedulerImpl: Removed TaskSet 0.0, whose tasks have all completed, from pool
16/03/17 14:29:11 INFO TaskSchedulerImpl: Stage 0 was cancelled
16/03/17 14:29:11 INFO TaskSetManager: Lost task 0.3 in stage 0.0 (TID 7) on executor 10.200.2.211: java.lang.ClassNotFoundException (top.spoofer.hbrdd.HbMain$$anonfun$1) [duplicate 7]
16/03/17 14:29:11 INFO DAGScheduler: ResultStage 0 (count at HbMain.scala:26) failed in 2.202 s
16/03/17 14:29:11 INFO TaskSchedulerImpl: Removed TaskSet 0.0, whose tasks have all completed, from pool
16/03/17 14:29:11 INFO DAGScheduler: Job 0 failed: count at HbMain.scala:26, took 3.172210 s
Exception in thread "main" org.apache.spark.SparkException: Job aborted due to stage failure: Task 1 in stage 0.0 failed 4 times, most recent failure: Lost task 1.3 in stage 0.0 (TID 6, 10.200.2.212): java.lang.ClassNotFoundException: top.spoofer.hbrdd.HbMain$$anonfun$1
	at java.net.URLClassLoader$1.run(URLClassLoader.java:366)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:270)
	at org.apache.spark.serializer.JavaDeserializationStream$$anon$1.resolveClass(JavaSerializer.scala:67)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1612)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1517)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1771)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1350)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:1990)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:1915)
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

Driver stacktrace:
	at org.apache.spark.scheduler.DAGScheduler.org$apache$spark$scheduler$DAGScheduler$$failJobAndIndependentStages(DAGScheduler.scala:1280)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$abortStage$1.apply(DAGScheduler.scala:1268)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$abortStage$1.apply(DAGScheduler.scala:1267)
	at scala.collection.mutable.ResizableArray$class.foreach(ResizableArray.scala:59)
	at scala.collection.mutable.ArrayBuffer.foreach(ArrayBuffer.scala:47)
	at org.apache.spark.scheduler.DAGScheduler.abortStage(DAGScheduler.scala:1267)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$handleTaskSetFailed$1.apply(DAGScheduler.scala:697)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$handleTaskSetFailed$1.apply(DAGScheduler.scala:697)
	at scala.Option.foreach(Option.scala:236)
	at org.apache.spark.scheduler.DAGScheduler.handleTaskSetFailed(DAGScheduler.scala:697)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.doOnReceive(DAGScheduler.scala:1493)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:1455)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:1444)
	at org.apache.spark.util.EventLoop$$anon$1.run(EventLoop.scala:48)
	at org.apache.spark.scheduler.DAGScheduler.runJob(DAGScheduler.scala:567)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1813)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1826)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1839)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1910)
	at org.apache.spark.rdd.RDD.count(RDD.scala:1121)
	at top.spoofer.hbrdd.HbMain$.main(HbMain.scala:26)
	at top.spoofer.hbrdd.HbMain.main(HbMain.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
Caused by: java.lang.ClassNotFoundException: top.spoofer.hbrdd.HbMain$$anonfun$1
	at java.net.URLClassLoader$1.run(URLClassLoader.java:366)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:270)
	at org.apache.spark.serializer.JavaDeserializationStream$$anon$1.resolveClass(JavaSerializer.scala:67)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1612)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1517)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1771)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1350)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:1990)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:1915)
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
16/03/17 14:29:11 INFO SparkContext: Invoking stop() from shutdown hook
16/03/17 14:29:11 INFO SparkUI: Stopped Spark web UI at http://192.168.88.16:4040
16/03/17 14:29:11 INFO DAGScheduler: Stopping DAGScheduler
16/03/17 14:29:11 INFO SparkDeploySchedulerBackend: Shutting down all executors
16/03/17 14:29:11 INFO SparkDeploySchedulerBackend: Asking each executor to shut down
16/03/17 14:29:11 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
16/03/17 14:29:11 INFO MemoryStore: MemoryStore cleared
16/03/17 14:29:11 INFO BlockManager: BlockManager stopped
16/03/17 14:29:11 INFO BlockManagerMaster: BlockManagerMaster stopped
16/03/17 14:29:11 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
16/03/17 14:29:11 INFO SparkContext: Successfully stopped SparkContext
16/03/17 14:29:11 INFO ShutdownHookManager: Shutdown hook called
16/03/17 14:29:11 INFO RemoteActorRefProvider$RemotingTerminator: Shutting down remote daemon.
16/03/17 14:29:11 INFO ShutdownHookManager: Deleting directory /tmp/spark-ecaf17d0-2bda-44f5-b45c-808a83e252b4
16/03/17 14:29:11 INFO RemoteActorRefProvider$RemotingTerminator: Remote daemon shut down; proceeding with flushing remote transports.
16/03/17 14:29:11 INFO RemoteActorRefProvider$RemotingTerminator: Remoting shut down.

Process finished with exit code 1

```

在运行spark程序时候抛出找不到主类的异常，这个时候是因为本地提交的任务没有将main函数的jar丢到workers上运行。
所以在workers上是找不到你的程序的！

### 配置intellij project 属性

#### 配置代码

新建 sbt project， 这个就不多说了。代码如下：

```
package top.spoofer.hbrdd

import org.apache.spark.{SparkContext, SparkConf}

object HbMain {
  private val master = "Core1"
  private val port = "7077"
  private val appName = "hbase-rdd_spark"
  private val data = "hdfs://Master1:8020/test/spark/hbase/testhb"

  def main(args: Array[String]) {
    val sparkConf = new SparkConf()
      .setMaster(s"spark://$master:$port")
      .setAppName(appName).setJars(List("/home/lele/coding/hbrdd/out/artifacts/hbrdd_jar/hbrdd.jar"))

    val sc = new SparkContext(sparkConf)
    val ret = sc.textFile(data).map({ line =>
      val Array(k, col1, col2, _) = line split "\t"
      val content = Map("col1" -> col1, "col2" -> col2)
      k -> content
    })

    println(ret.count())

    sc.stop()
  }
}
```

工程包结构如下图：

![project-code.png][1]

刚才说了是因为我们的工程jar包没有被提交到worker上，所以我们可以手动将我们的jar加入到要提交jar list里去。
使用SparkConf的setJars加入jar， 代码如下：

```
//其中jarPath是你打包出来的jar，打包操作下面有述
val jarPath = "/home/lele/coding/hbrdd/out/artifacts/hbrdd_jar/hbrdd.jar"
val sparkConf = new SparkConf().setMaster(s"spark://$master:$port").setAppName(appName)
.setJars(List(jarPath))

```
这样提交job的时候我们的jar就一起被提交了。

现在代码配置好了，需要配置 intellij，使其帮我们把项目打包成jar。具体步骤如下：

#### 创建Artifacts

点击：File->project structure->Artifacts，添加一个Artifacts， 如下图：

![new-artif.png][2]

#### 配置创建好的Artifacts

不多说，看下图：

![create-artif.png][3]

#### 去除已有的依赖

因为集群上已经有scala的lib和spark assembly的lib，所以这里去除这些依赖。如果没有就不用了。
如果把这些依赖都打包成jar， 那么这个jar会非常大！因为spark assembly就有100多M。

如图：

![ref-remove.png][4]

到此，应该可以正确地运行程序了。

[1]: http://www.spoofer.top/assets/images/2016/03/project-code.png
[2]: http://www.spoofer.top/assets/images/2016/03/new-artif.png
[3]: http://www.spoofer.top/assets/images/2016/03/create-artif.png
[4]: http://www.spoofer.top/assets/images/2016/03/ref-remove.png
