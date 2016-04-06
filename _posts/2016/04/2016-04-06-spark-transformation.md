---
layout: post
title:  "spark 中的常用的transformation操作简单总结"
date:   '2016-04-05 7:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'spark'
excerpt: 'spark 中的常用的transformation操作简单总结'
keywords: 'spark transformation'
---
### 概述

本文简单总结spark中transformation操作，如：map、join、cogroup等。

<!--maore-->

### map(func)

使用函数func对每个元素进行处理，得到新的元素组成。返回一个新的分布式数据集。

### fliter(func)

使用函数func对每个元素进行处理后得到结果为true的元素，返回一个新的分布式数据集。

### flatMap(func)

一个类似于map的函数，每个输入的元素会被映射到0个或者多个输出元素，所以flatMap返回的值的是一个seq而不是简单的一个元素。

### mapPartitions(func)

一个类似与map的函数，对RDD的每个分区起作用，如果RDD的元素类型为T的话，那么func的类型一定要是Iterator[T] => Iterator[U]。

### mapPartitionsWithIndex(func)

其与mapPartitions(func)类似， 但是func带有一个整数参数表示分区的索引值，如果RDD的元素类型为T的话，那么func的类型一定要是Iterator[T] => Iterator[U]。

### sample(withReplacement, fraction, seed)

根据随机种子seed，随机抽样出数量为fraction的数据。

### pipe(command, [envVars])

通过管道的方式对RDD的每个分区使用shell指令进行操作，返回对应的结果。

### union(otherDataset)

返回一个新的数据集，由元数据集和参数联合而成。

### intersection(otherDataset)

返回两个数据RDD的交集。

### distinct([numTasks])

数据去重，返回RDD中包含原数据集所有不重复元素的新的数据集。

### groupByKey([numTasks])
