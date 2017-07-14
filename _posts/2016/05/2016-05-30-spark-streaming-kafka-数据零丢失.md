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
接收器将接收到的数据存储在spark中，进行热备，说白了就是将数据保存到两个（默认情况）Executor的内存中以进行容错。
一旦数据被存储到spark中，接收器就对其进行确认操作（更新kafka的偏移量到zookeeper）。
这样可以保证在接收器（Executor）突然挂掉后也不会丢失数据，因为数据即使被接收了，但还没有被持久化的情况下是不会进行确认的。
所以在Executor恢复的时候可以重新读取数据这是基于kafka支持重放机制。

![可靠数据源和接收器.png][1]

#### 数据的元数据持久化

可靠的数据源和可靠的接收器可以使我们从接收器或者Executor突然挂掉的情况下正常恢复。但如果driver挂掉的话，问题就更复杂了。
有很多的技术可以让driver从失败中恢复，其中一个就是对数据的元数据进行checkpoint。进行checkpoint的时候，一般会将数据持久化到可靠的文件系统中，
例如S3、HDFS等。这样driver可以利用这些持久化的数据从失败中进行恢复。

![checkpoint.png][2]


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

#### 数据丢失

即使有了上述的两个条件来保证数据不丢失，但还是存在数据丢失的场景。例如：


1、Executor 从kafka接收到数据后将数据备份、缓存到其他Executor的内存中，进行热备。

2、接收器通知数据源数据已经接收完毕，其实就是更新kafka 的 offset。

3、driver提交了jobset，并且Executor 开始这些job处理数据。

4、driver突然挂掉。因为driver挂掉，相应的Executor也会被杀死。

5、因为数据缓存在多个Executor的内存中，所以Executor挂掉，缓存的数据也被丢失了，
而且这些数据已经被确认接收、消费，所以无法在本地和源端进行复原，造成数据丢失。

因为driver的突然退出而造成的数据丢失实在可怕，必须有一个解决方案。

#### WAL

对应上述可能造成数据丢失的场景， spark 1.2 后开始引入了WAL (Write ahead log) 机制。

开启了WAL后，接收器将接收到的数据写入到容错的存储系统中，例如HDFS、S3等。
这样即使driver失败了，还是可以在恢复中找到丢失的数据，因为已经保存在容错的存储系统中。
因为接收到数据后需要写入容错的存储系统，这难免会损失性能。

![checkpointAndWAL.png][3]

#### At Least Once 语义

即使WAL确保数据不会丢失，但是在spark streaming 整合kafka的情况下，还是难以保证exactly once语义的。
如以下场景：

1、接收器接收到数据，并且对数据进行WAL。

2、但在接收器更新zookeeper中的kafka偏移量之前突然挂掉了。
因为没有更新kafka的offset，所以在源端数据是没有被消费的。但此时数据已经被写入到WAL了。

3、接收器重新恢复，并且处理WAL中没有被处理的数据。

4、处理完WAL的数据后，接收器开始从源端kafka消费数据。因为offset没有被更新，所以恢复时处理过的数据重新在kafkak源端再次被读取。
这样这部分的数据被处理了2次。

#### Direct API

开启WAL会有损性能，所以在spark 1.3中就引入了 Kafka direct API。Exectuor直接从kafka对应的topic中的分区消费数据。
基于Direct的方式，有spark streaming直接管理offset，可以给定offset范围，直接读取kafka分区的数据，
保证了每个offset范围只属于一个batch，从而保证了exactly-once 语义。
Direct API 说白其实就是当kafka topic是一个文件，像读取文件一样消费数据。

新的Direct API有点有：

1、不需要接收器，Exectuor直接使用 kafka 的简单消费者API消费数据。

2、不需要开启WAL，仍然可以从失败中恢复数据。避免了WAL对性能的损失。

3、保存了exactly-once语义。



[1]: http://www.spoofer.top/assets/images/2016/05/可靠数据源和接收器.png
[2]: http://www.spoofer.top/assets/images/2016/05/checkpoint.png
[3]: http://www.spoofer.top/assets/images/2016/05/checkpointAndWAL.png
