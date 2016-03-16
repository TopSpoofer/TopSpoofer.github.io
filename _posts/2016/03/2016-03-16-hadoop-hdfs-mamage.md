---
layout: post
title:  "hadoop hdfs 管理"
date:   '2016-03-16 14:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'hadoop'
excerpt: 'hadoop hdfs 管理'
keywords: 'hadoop hdfs 管理'
---

### 概述
本文主要简单介绍hdfs的管理和其中一些原理。

<!--more-->

### 永久性数据结构

不论是集群管理员还是hadoop开发人员， 深入了解namenode、
辅佐namenode和datanode等hdfs组件如何在磁盘上组织永久性数据结构是非常重要的。

#### namenode 的目录结构

在namenode被格式化后， 将产生如下目录结构：

```
${dfs.name.dir}/
 |------ current/
          |------- edits
          |------- fsimage
          |------- fstime
          |------- VERSION
```

dfs.name.dir 描述了一组目录， 各个目录存储着镜像内容。
这个机制是的系统具有一个复原能力，特别是当其中一个目录是NFS的一个挂载时。

其中VERSION文件是一个java属性文件， 包括正在运行的HDFS的版本信息，其内容如下：

```
#Fri Feb 26 11:51:29 CST 2016
namespaceID=1733371526
clusterID=CID-0409092d-3b7c-4e1f-9e04-b191649a8ae6
cTime=0
storageType=NAME_NODE
blockpoolID=BP-768146364-192.168.122.3-1455938204134
layoutVersion=-60
```
属性layoutVersion是一个负整数， 描述了HDFS持久化数据结构的版本，也称为布局。
需要注意的是，这个版本跟hadoop 发布包的版本没有一丝关系！只要布局变更，版本号就会递减（每次递减1），此时hdfs也需要升级。
否则磁盘仍然使用旧的版本的布局，新版本的namenode和datanode就无法正常工作了。

属性namespaceID 是文件系统的唯一标识符号，是在文件系统首次格式化时设置的。
任何的datanode在注册到namenode前都无法知道namespaceID的值，所以namenode可以使用这个ID鉴别新建的datanode。

cTime是namenode存储系统的创建时间。对刚格式化的存储系统，这个值为0。但在文件系统升级后该值会更新到新的时间戳。

storageType说明这个存储目录包含的是namenode的数据结构。

目录中还包括edits、fsimage、stime等二进制文件。这些文件都是使用hadoop的writable对象作为其序列化格式。

#### 文件系统映像（fsimage）和编辑日志（edits）

文件系统客户端进行写入操作的时候，这些操作先是被记录到编辑文件里。
namenode在内存中维护文件系统的元数据（当hdfs中文件非常多的数据这些元数据会非常大），当编辑日志被修改时相关的元数据也要同步更新。
而维持在内存中的元数据可以支持客户端的读写请求。

在每次执行写操作后， 且向客户端发送成功码前，编辑日志都需要更新和同步。
如果namenode要向多个目录进行写入操作，只有在写入操作都执行完毕后才可以返回成功码，以确保任何操作都不会因为机器故障而丢失。

fsimage 是文件系统的元数据的一个持久化检查点。并非每一个写入操作都会更新这个文件,
因为fsimage可能非常大,如果频繁进行写入操作会非常耗费系统资源。
而这个特性不会降低系统的恢复能力，因为如果namenode发生故障，
可以先把fsimage先装载到内存里重新构建新近的元数据，然后再执行编辑日志记录中记录的各项操作。
而namenode在启动阶段正是这样做的。
