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

上述ShuffleMapTask写的结果会给ResultTask去读。spark会`使用两种方式来读这些数据。
