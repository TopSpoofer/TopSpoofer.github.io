---
layout: post
title:  "spark TaskScheduler 简介"
date:   '2016-04-27 19:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'spark'
excerpt: 'spark TaskScheduler 简介'
keywords: 'spark TaskScheduler'
---

### 概述

本文主要讲述spark 中的TaskScheduler的作用。

<!--more-->

#### TaskScheduler 作用简介

TaskScheduler 本身是一个任务调度的接口, 通过不同的 SchedulerBackend 对任务进行具体的调度.
TaskScheduler 主要接收 DAGScheduler 提交的 TaskSet 然后提交到集群中进行执行。若某个任务失败, 则根据重试的规则进行重新运行, 并且讲执行的结构反馈给 DAGScheduler. TaskScheduler 中也实现了任务推测执行的机制, 并且 TaskScheduler为每一个TaskSet维护了一个TaskSetManager用于追终错误信息, 结果和本地化等信息.

#### TaskScheduler 实现类型列表

根据运行模式的不同, spark提供了多种 TaskScheduler 的实现. TaskScheduler 的实现列表如下:

![taskScheduler.png][1]

下面只对Standalone模式进行讨论. 所以只会对 TaskSchedulerImpl 原理进行说明.


#### Standalone 下的 TaskSchedulerImpl

TaskSchedulerImpl 的初始化和启动是在SparkContext 中进行的，初始化的时候会传入 SparkDeploySchedulerBackend 对象，
启动则调用start方法， 在start方法中判断是否启动任务的推测执行，其由spark.speculation属性指定。

而SparkDeploySchedulerBackend中的start方法也是在TaskSchedulerImpl的start方法中调用的。在这个方法中会初始化AppClient对象。主要还是用于与Driver和master的Akka actor进行通信和交互、注册spark Application等。

DAGScheduler处理job的最后一步是将TaskSet提交给TaskScheduler进行调度的，调用的是TaskScheduler中的submitTasks方法。


[1]: http://www.spoofer.top/assets/images/2016/04/taskScheduler.png
