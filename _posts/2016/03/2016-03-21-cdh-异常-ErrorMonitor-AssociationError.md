---
layout: post
title:  "cdh 5.5.1 莫名其妙的异常--ERROR ErrorMonitor: AssociationError"
date:   '2016-03-21 14:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'cdh'
excerpt: 'cdh 5.5.1 莫名其妙的异常--ERROR ErrorMonitor: AssociationError'
keywords: 'cdh-5.5.1 异常 ERROR ErrorMonitor: AssociationError'
---

### 概述

前几天，在cdh 5.5.1的下提交任务，出现了一个莫名其妙的错误信息，但是不影响程序执行。其报错如下：
ERROR ErrorMonitor: AssociationError [akka.tcp://sparkDriver@10.16.255.231:46792] <- [akka.tcp://sparkExecutor@10.16.255.213:48822]: Error [Shut down address:

<!--more--->

### 错误描述

```
16/03/19 10:40:58 INFO SparkUI: Stopped Spark web UI at http://10.16.255.231:4040
16/03/19 10:40:58 INFO DAGScheduler: Stopping DAGScheduler
16/03/19 10:40:58 INFO SparkDeploySchedulerBackend: Shutting down all executors
16/03/19 10:40:58 INFO SparkDeploySchedulerBackend: Asking each executor to shut down
16/03/19 10:40:58 ERROR ErrorMonitor: AssociationError [akka.tcp://sparkDriver@10.16.255.231:46792] <- [akka.tcp://sparkExecutor@10.16.255.213:48822]: Error [Shut down address: akka.tcp://sparkExecutor@10.16.255.213:48822] [
akka.remote.ShutDownAssociation: Shut down address: akka.tcp://sparkExecutor@10.16.255.213:48822
Caused by: akka.remote.transport.Transport$InvalidAssociationException: The remote system terminated the association because it is shutting down.
]
akka.event.Logging$Error$NoCause$
16/03/19 10:40:58 ERROR ErrorMonitor: AssociationError [akka.tcp://sparkDriver@10.16.255.231:46792] <- [akka.tcp://sparkExecutor@10.16.255.211:60743]: Error [Shut down address: akka.tcp://sparkExecutor@10.16.255.211:60743] [
akka.remote.ShutDownAssociation: Shut down address: akka.tcp://sparkExecutor@10.16.255.211:60743
Caused by: akka.remote.transport.Transport$InvalidAssociationException: The remote system terminated the association because it is shutting down.
]
akka.event.Logging$Error$NoCause$
16/03/19 10:40:58 ERROR ErrorMonitor: AssociationError [akka.tcp://sparkDriver@10.16.255.231:46792] <- [akka.tcp://sparkExecutor@10.16.255.212:35300]: Error [Shut down address: akka.tcp://sparkExecutor@10.16.255.212:35300] [
akka.remote.ShutDownAssociation: Shut down address: akka.tcp://sparkExecutor@10.16.255.212:35300
Caused by: akka.remote.transport.Transport$InvalidAssociationException: The remote system terminated the association because it is shutting down.
]
akka.event.Logging$Error$NoCause$
16/03/19 10:40:58 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
16/03/19 10:40:58 INFO MemoryStore: MemoryStore cleared
16/03/19 10:40:58 INFO BlockManager: BlockManager stopped
16/03/19 10:40:58 INFO BlockManagerMaster: BlockManagerMaster stopped
16/03/19 10:40:58 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
16/03/19 10:40:58 INFO SparkContext: Successfully stopped SparkContext
16/03/19 10:40:58 INFO RemoteActorRefProvider$RemotingTerminator: Shutting down remote daemon.
16/03/19 10:40:58 INFO RemoteActorRefProvider$RemotingTerminator: Remote daemon shut down; proceeding with flushing remote transports.
16/03/19 10:40:58 INFO ShutdownHookManager: Shutdown hook called
16/03/19 10:40:58 INFO ShutdownHookManager: Deleting directory /tmp/spark-2d164edb-30bb-499a-a64b-0bdb043ba513
16/03/19 10:40:58 INFO Remoting: Remoting shut down
16/03/19 10:40:58 INFO RemoteActorRefProvider$RemotingTerminator: Remoting shut down.
```

错误信息如上，非常奇怪，在很多群里问了都没有得到答复。而且发现不只是提交任务的时候发生错误，在spark-shell里都出现这个错误。
本来因为代码问题，但发现只是一个count的程序，不可能出错！而spark-shell都有问题的情况下，只能说明是cdh集群配置有问题或者cdh上游有问题了。

经过查询，更加相信是cdh的问题。

http://www.cloudera.com/documentation/enterprise/latest/topics/cdh_rn_spark_ki.html
 这里有说明。

cdh社区的解决方案是： http://www.cloudera.com/documentation/enterprise/latest/topics/spark_applications_configuring.html?scroll=concept_ws3_5t1_mt_unique_1

但我根据教程进行配置还是没有效果，而且发现，这是很脑残的解决方案！要解决这个还是等更新或者补丁吧！
