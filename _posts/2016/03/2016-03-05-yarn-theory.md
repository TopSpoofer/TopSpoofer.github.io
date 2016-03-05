---
layout: post
title:  "hadoop 2.6 Yarn 原理"
date:   '2016-03-05 14:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'hadoop'
excerpt: 'hadoop 2.6 Yarn 原理'
keywords: 'hadoop,2.6,Yarn,大数据'
---

### 概述

本文简单地讲述hadoop mapreduceV2 框架 yarn的工作原理。由于mapreduceV1框架在大集群中面临着扩展性瓶颈，所以2010年雅虎团队开始设计了mapreduceV2框架Yarn (yet another resource negotiaor)。


### yarn框架图

来源于官网的yarn架构图：

![yarn_architecture.gif][1]

如上图，yarn将原本mrV1中JobTrack的的职能划分为多个独立的实体，包括管理集群的资源的资源管理器和管理集群上运行任务生命周期的应用管理器。
应用服务器和资源管理器协商分配集群的计算资源：容器。容器是yarn为集群资源进行隔离而提出来的框架，容器是一个动态分配的资源单位，
其将cpu、内存、磁盘、网络等资源封装成为一个实体。每一个任务对应一个容器，所以任务只能在对应的容器里跑的。
容器由集群节点上运行的节点管理器进行监视，确保应用程序使用的资源不会超过其被分配的资源。
所以yarn框架里有3个重要的M分别是：

```
RM      Resource Manager        资源管理器
AM      Application Master      应用程序管理器
NM      Node Manager            节点管理器
```

yarn比mrV1包括更多的实体：

```
*   提交mapreduce作业的客户端
*   yarn资源管理器， 负责协调集群上计算资源的分配
*   yarn节点管理器， 负责启动和监视集群中机器上的计算容器。
*   应用程序管理器， 负责协调运行mapreduce作业的任务， 其和mapreduce任务都在容器中运行， 这些容器由资源管理器分配并且由节点管理器进行管理。
*   hdfs， 用来和其他实体间共享作业文件。
```

下面分别讲解这3M。


### 资源管理器 RM

RM是一个全局的资源管理器，其负责整个系统资源的管理，和AM协调应用程序资源的分配。RM主要分为调度器和ASM（Applications Manager）。

#### 调度器

调度器根据系统状态将系统中的资源分配给正在运行的应用程序，调度器只关注系统调度工作

[1]: http://spoofer.top/assets/images/2016/03/yarn_architecture.gif
