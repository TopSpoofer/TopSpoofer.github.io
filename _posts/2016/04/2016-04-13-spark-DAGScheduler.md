---
layout: post
title:  "spark DAGScheduler 简介"
date:   '2016-04-13 19:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'spark'
excerpt: 'spark DAGScheduler'
keywords: 'spark DAGScheduler'
---

### 概述

本文主要讲述spark中的DAGScheduler。

<!--more-->

##### DAGScheduler功能简介

DAGSchedulerpo是面向stage层的调度器，负责接受用户提交的job。根据rdd的依赖关系划分出不同的stage，并且在每一个stage内封装好taskset，然后结合当前的缓存情况和数据就近原则将taskset提交给TaskScheduler。

SparkContext作为整个spark程序的入口，不管是spark、spark streaming、spark sql都需要先创建一个SparkContex对象，然后基于这个对象进行后续的rdd操作。而SaprkContex的主要功能就是初始化SparkUI、创建DAGScheduler、TaskScheduler。

其中TaskScheduler是接收Task和分到的资源并且进行维护。


##### DAGScheduler处理job的提交

```
1、Appliction代码中的Spark Action操作将调用sc.runJob方法。

2、SparkContext中的runJob方法被调用。

3、DAGScheduler中的runJob方法被调用。

4、DAGScheduler中的submitJob被调用，准备Job的提交，进而触发JobSubmit事件。

5、DAGScheduler调用handleJobSubmitted方法进行事件处理，handleJobSubmitted做的第一件事就是调用newStage方法来生成新的stage。

6、生成stage后，handleJobSubmitted继续执行，并且生成ActiveJob。

7、如果是简单的job，没有依赖关系并且只有一个分区， 该类型的job会使用local thread处理而不是提交到TaskScheduler上处理。非local运行的job会继续进行stage的提交。

8、计算stage之间的依赖关系（stage DAG），并且对依赖关系进行处理。其实就是从后向前寻找父stage。如果没有父stage， 那么说明这个stage所有的依赖已经准备完毕，提交task，并且将当前的stage放入runningStages中。如果有父stage，则先提交父stage， 把当前stage放入等待队列里。

9、任务提交主要是由submitMissingTasks方法来完成的。主要是根据当前stage所依赖的rdd的分区分布来产生与分区数量相同的task。如果是MapStage则产生ShuffleMapTask， 否则产生ResultTask。task的位置都是经过getPreferredLocs方法极端得来的最佳位置。

10、将整个TaskSet提交到TaskScheduler进行调度执行。
```
