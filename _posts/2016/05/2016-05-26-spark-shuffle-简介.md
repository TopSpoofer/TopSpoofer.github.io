---
layout: post
title:  "spark shuffle 简介"
date:   '2016-05-26 19:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'spark'
excerpt: 'spark shuffle 简介'
keywords: 'spark shuffle'
---

### 概述

本文主要介绍spark的shuffle。

<!--more-->

### shuffle 分析

shuffle描述的是一个过程，表现的是多对多的依赖关系。在mapreduce中，shuffle链接了map阶段和reduce阶段。
每个reduce task都会从map task产生的数据里读取其中的一片数据，shuffle通常分为map阶段的数据和reduce阶段的数据副本，
这样在极端的情况下可能会触发map task数目 × reduce task数目 个数据副本。
在map阶段需要根据reduce阶段的task数量来决定么每个map task输出的数据分片数，这些数据可能保存在磁盘或内存里面。
这些分片可能是每一个分片就一个文件或者多个分片一个文件，外加一个索引来记录每个分片的位置。
spark有些task间数据流转是不需要通过shuffle的，比如rdd之间是窄依赖的情况下，不需要shuffle。但是大多数情况下task间还是需要通过shuffle来传递数据的。


#### shuffle 写

spark中shuffle输出的ShuffleMapTask会为每个ResultTask创建对应的Bucket，ShuffleMapTask产生的结果会根据设置的partitioner得到对应的BucketId，
然后填充到相应的Bucket中去。每个ShuffleMapTask的输出结果可能包含所有的ResultTask所需要的数据，所以每个ShuffleMapTask创建Bucket的数目是和ResultTask的数目相等的。

ShuffleMapTask创建的Bucket对应磁盘上的一个文件，此文件被称为BlockFile。通过spark.shuffle.file.buffer.kb 属性配置德尔缓冲区就是用来创建FastBufferOutputStream输出流的。
如果在配置文件中设置了spark.shuffle.consolidateFiles属性为true，那么ShuffleMapTask产生的Bucket就不一定单独对应一个文件了，而是对应文件的一部分，这样做会大量减少产生的BlockFile文件的数量。

ShuffleMapTask在某个节点上第一次执行时，会为每个ResultTask创建一个输出文件。
并且把这些文件组成ShuffleFileGroup，当这个ShuffleMapTask执行完后，当前创建的ShuffleFileGroup可以释放掉，进行循环使用。
当又有ShuffleMapTask在这个节点上执行时，不需要创建新的输出文件，而是在上次的ShuffleFileGroup中已经创建的文件里追加写一个Segment。
如果当前的ShuffleMapTask还没有执行完，此时又在此节点上启动了新的ShuffleMapTask， 那么新的ShuffleMapTask只能又创建新的输出文件再组成一个ShuffleFileGroup来进行结果输出。


#### shuffle 读

上述ShuffleMapTask写的结果会给ResultTask去读。spark会`使用两种方式来读这些数据。spark使用两种方式去读取这些数据：使用普通的socket方式、使用netty框架。
如果要使用netty框架方式来读取数据，可以通过配置spark.shuffle.use.netty为true来启动。

ResultTask读取数据时会通过BlockManager根据BlockId把相关的数据返回给ResultTask。如果是netty框架，blockManage会创建ShuffleSender 来专门负责发送数据。
如果ResultTask需要偶的数据在本节点上，那么就直接读磁盘即可，不需要通过网络获取。
这一点要比mapreduce要好，因为mapreduce读取数据时，即使数据在本地，也会通过网络获取。

spark的shuffle过程的数据都是没有经过排序的，这一点比mapreduce框架节省很多时间。
ResultTask读取的数据先放在HashMap里，如果数据量小，占用的内存小，如果内存不够会根据spark.shuffle.spill的设置，可能直接失败或者把数据写到磁盘。

#### shuffle 相关配置属性

##### spark.shuffle.consolidateFiles

默认值： false

说明：
如果为true，在shuffle时就合并中间文件，对于有大量Reduce任务的shuffle来说，合并文件可以提高文件系统性能。


##### spark.shuffle.spill

默认值： true

说明：
如果为true，在shuffle期间通过一溢出数据到磁盘来降低内存的使用量。溢出的值由spark.shuffle.memory.Fraction指定。

##### spark.shuffle.spill.compress

默认值：true

说明：
是否压缩在shuffle期间溢出的数据， 如果压缩数据，讲使用spark.io.compression.codec进行压缩。


##### spark.shuffle.file.buffer.kb

默认值： 100

说明：
每个shuffle的文件输出流内存缓冲区的大小，以kb为单位。

##### spark.reduce.maxMbInFlight

默认值: 48

说明：
每个reduce任务同时获取map输出的最大值，以M为单位。由于每个map输出都需要一个缓冲区来接收它，这样每个reduce任务有着固定的内存开销，所以要设置小点，除非有很大的内存。
