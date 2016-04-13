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

DAGScheduler是面向stage层的调度器，负责接受用户提交的job。根据rdd的依赖关系划分出不同的stage，并且在每一个stage内封装好taskset，然后结合当前的缓存情况和数据就近原则将taskset提交给TaskScheduler。

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

7、
```
