---
layout: post
title:  "利用spark streaming将kafka topic 数据导入到hbase"
date:   '2016-04-14 14:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'spark'
excerpt: '利用 spark streaming 将 kafka topic 数据导入到 hbase'
keywords: 'kafka 数据 spark streaming 导入 hbase saveAsNewAPIHadoopDataset'
---

### 概述

本文主要讲述如何将 kafka topic 的数据经spark streaming 导入到hbase. 本文的Example使用scala开发,如果你是写java的朋友,那可能会有点为难了.

可能网上已经有很多教程了, 其中也可能有很多的教程都是使用c/s模式访问hbase, 本文不是使用c/s模式访问hbase的.

<!--more-->

#### 开发前准备

安装hadoop, zookeeper, spark, hbase, kafka集群

因为我是使用cdh管理集群的, 安装cdh的教程请参见: (cdh-5.5.1 安装)[http://www.spoofer.top/2016/03/04/centos6-install-cdh]

kafka集群没有使用cdh的,所以是独立安装的. 安装kafka请参见: (kafka 0.9.0.1 安装)[http://www.spoofer.top/2016/04/08/kafka-%E5%88%86%E5%B8%83%E5%BC%8F-install]

创建kafka topic: users, 并且测试本地是否可以生产消息和消费消息.


#### 创建工程和导入需要的库

使用idea创建工程, 在工程的根目录下创建lib目录. 将spark-assembly-1.5.1-hadoop2.6.0.jar移到lib里, 因为我的集群使用的spark的版本是1.5.1的.

这里没有使用sbt来管理spark的库, 如果你需要, 可以修改build.sbt的spark的版本.或者加入spark 的mvn依赖.

导入hbrdd, hbrdd项目参见: [hbrdd project](https://github.com/TopSpoofer/hbrdd)
这里有详细的安装,使用教程.


#### 使用idea本地提交spark程序到远程集群运行

如何在idea上打包并提交到集群上运行请参见: [idea本地提交spark程序到远程集群运行](http://www.spoofer.top/2016/03/16/intellij%E8%BF%9C%E7%A8%8B%E6%8F%90%E4%BA%A4%E4%BB%BB%E5%8A%A1%E5%88%B0spark%E9%9B%86%E7%BE%A4)


#### 项目地址和源码

project src: (code)[https://github.com/TopSpoofer/kafka-spark-streaming-to-hbase]
