---
layout: post
title:  "spark streaming + kafka 数据零丢失的原理"
date:   '2016-05-30 19:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'spark'
excerpt: 'spark streaming kafka 数据零丢失'
keywords: 'spark streaming kafka 数据零丢失'
---


### 概述

本文主要介绍 spark streaming + kafka 是如何保证数据零丢失的。

在我们部署好spark streaming 和 kafka 后就可以使用spark streaming提供的保证数据零丢失的机制，
但需要注意的是数据零丢失并不代表数据 exactly once 语义，exactly once 语义是很难保证的，后面会有文章介绍。

<!--more-->

#### 先决条件

要保证spark streaming + kafka保证零数据丢失，需要下面几个条件：

```
1、可靠的数据源和可靠的数据接收器。基于kafka的原理，所以kafaka这个数据源是可靠的。
2、应用程序的meta 数据需要被 driver 持久化的， 这个其实就是checkpoint。
3、需要启用WAL。
```

#### 可靠的数据源和可靠的接收器

对于kafka，有一个很对消息队列的都没有的一个功能就是重放。所以spark streaming可以对已经接收的数据进行记录和确认。
输入的数据首先被接收器接收，接收器是在Executor上进行的，这里先不讨论接收器是如何做的。
接收器将接收到的数据存储在spark中，进行热备，说白了就是将数据保存到两个（默认情况）Executor中以进行容错。
一旦数据被存储到spark中，接收器就对其进行确认操作（更新kafka的偏移量到zookeeper）。
这样可以保证在接收器（Executor）突然挂掉后也不会丢失数据，因为数据即使被接收了，但还没有被持久化的情况下是不会进行确认的。
所以在Executor恢复的时候可以重新读取数据这是基于kafka支持重放机制。

![可靠数据源和接收器.png][1]

#### 数据的元数据持久化

可靠的数据源和可靠的接收器可以使我们从接收器或者Executor突然挂掉的情况下正常恢复。但如果driver挂掉的话，问题就更复杂了。
有很多的技术可以让driver从失败中恢复，其中一个就是对数据的元数据进行checkpoint。进行checkpoint的时候，一般会将数据持久化到可靠的文件系统中，
例如S3、HDFS等。这样driver可以利用这些持久化的数据从失败中进行恢复。

##### checpoint 的内容

spark streaming 会checkpoint 两种类型的数据，其中一种就是上述的metadata checkpoint，另外一种是datacheckpoint。

1、metadata checkpoint

metadata checkpoint 保存了定义streaming 计算逻辑到可靠的支持容错的存储系统，用来恢复driver。其中元数据包括

```
1、配置： 用来创建该streaming application 的所有配置。
2、代码： 其实就是DStream操作。
3、还未完成的batchs：被提交了但没有执行或者没有完成的batchs。
```

2、data checkpoint

data checkpoint 是保存生成的RDDs到可靠的容错的存储系统。这在某些statefull转换中是需要的。在这种转换中，生成 RDD 需要依赖前面的 batches，会导致依赖链随着时间而变长。为了避免这种没有尽头的变长，要定期将中间生成的 RDDs 保存到可靠存储来切断依赖链。







































[1]: http://www.spoofer.top/assets/images/2016/05/可靠数据源和接收器.png
