---
layout: post
title:  "spark 中的常用的action操作简单总结"
date:   '2016-04-06 19:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'spark'
excerpt: 'spark 中的常用的action操作简单总结'
keywords: 'spark action'
---

### 概述

本文主要简单介绍spark中的action算子，如reduce、count等。

<!--more-->

#### reduce(func)

通过函数func对数据集中的元素进行聚集。这个函数func必须是关联性的，以确保可以被正确地并发执行。

#### collect()

在driver的程序当中，以数组的形式返回数据集的所有元素。这个通常会在使用filter等操作后返回一个足够小的数据子集。

#### count()

计算数据集的元素个数。

#### first()

返回数据集的第一个元素，类似于take(1)。

#### takeSample(withReplacement, num, seed)

在数据集中随机采样num个元素组成一个数组返回。可以选择是否用随机数替换不足的部分，seed用于指定的随机数生成器种子。

#### takeOrdered(n, [ordering])

数据集排序后的limit(n)。

#### saveAsTextFile(path)

将数据集的元素一text的形式保存到本地文件系统、hdfs、获取其他hadoop支持的文件系统。spark会调用元素的toString方法，并且将其转换为文件的一行。

#### saveAsSequenceFile(path)

将数据集的元素以sequenece的格式保存到指定的目录下，可以是本地文件系统、hdfs或者hadoop支持的文件系统。RDD元素必须是key-value
组成，并且实现了hadoop的Writable接口，或者可以隐式转换为Writable。

#### saveAdObjectFile(path)

使用java的序列化方法保存到本地文件系统，可以被SparkContext.objectFile()加载。

#### countByKey()

对(k, v)类型的RDD有效，返回一个(k, Int)对的Map，表示每一个key对应的元素的个数。

#### foreach(func)

在数据集上对每一个元素进行调用func函数。通常用于更新一个累加器变量或者和外部系统进行交互。
