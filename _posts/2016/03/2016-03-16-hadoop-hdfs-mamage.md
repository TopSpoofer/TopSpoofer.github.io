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

##### fsimage

fsimage 是文件系统的元数据的一个持久化检查点。并非每一个写入操作都会更新这个文件,
因为fsimage可能非常大,如果频繁进行写入操作会非常耗费系统资源。
而这个特性不会降低系统的恢复能力，因为如果namenode发生故障，
可以先把fsimage先装载到内存里重新构建新近的元数据，然后再执行编辑日志记录中记录的各项操作。
而namenode在启动阶段正是这样做的。

fsimage文件是系统的元数据， 这些元数据是指文件系统中所有的目录和文件inode的序列化信息。
inode其实是文件的元数据的内部描述方式，就是文件的信息了，如size、访问时间等。

数据快存储在datanode中，而fsimage并不描述datanode的而是namenode将这种映射关系放在内存里。
当datanode加入集群时，namenode会向datanode索取块列表用来建立映射关系除了这些，
namenode还会定期向datanode征询，以确保拥有最新的映射关系。

##### edist

edist文件会无限增长，虽说这种情况下，这对于namenode的运行没有如何影响，
但在重启的时候需要恢复编辑日志的各项操作，这个时候文件系统会处于离线状态！

基于这个问题，hadoop的解决方案是运行一个辅助namenode为内存中的文件系统元数据创建检查点。如下图：

![edist-adn-fsimage.png][1]


创建检查点的步骤如下：

```
  1、辅助namenode请求主namenode停止使用edist文件，将新的写操作写到新的文件里。
  2、辅助namenode使用http get从主namenode中获取需要的fsimage和edist文件。
  3、辅助namenode将获取到的fsimage和edist文件载入到内存里面，逐一执行edist中的操作，并且创建新的fsimage文件。
  4、辅助namenode利用http post将新的fsimage发送到主namenode。
  5、主namenode用辅助namenode post过来的fsimage文件替换旧的fsimage文件；
    使用步骤一的新的edist文件替换旧的edist文件，还更新fsimage文件来记录检查点的执行时间。
```

经过上面5个步骤，主那么弄得就拥有了最新的fsimage和新的edist文件。如果当namenode处于安全模式时，
系统管理员还可以调用hadoop dfsadmin -saveNamespace命令来创建检查点。

这个过程清晰地解析了辅助namenode和主namenode需要拥有相近的内存的原因----辅助namenode需要将fsimage载入到内存里面。
所以在大型的集群里辅助namenode需要运行在一台专用的机器上。

创建检查点的触发条件受两个参数控制。通常情况下，辅助namenode每隔一个小时（由fs.checkpoint.period属性设置，以秒为单位）创建检查点。
当编辑日志的大小达到64M(由fs.checkpoint.size属性设定， 单位字节)时，即使没有达到fs.checkpoint.period，也会创建一个检查点。
而系统会每5分钟就对edist的大小进行检查。


#### 辅助namenode的目录结构

创建检测点的过程不仅为主namenode创建检查点数据，还使辅助namenode最终也拥有一份检查点的数据。存储在（previous.checkpoint子目录下）。
这份数据可以用作辅助namenode的备份元数据即使不是最新的。

辅助namenode的目录结构：

```
$(fs.checkpoint.dir)/
  |------- current/
  |         |------- VERSION
  |         |------- edist
  |         |------- fsimage
  |         |------- fstime
  |
  |------- previous.checkpoint/
            |------- VERSION
            |------- edist
            |------- fsimage
            |------- fstime
```

可以看到，辅助namenode的previous.checkpoint、current目录的布局和主namenode的current布局相同。
这种设计的好处是，在主namenode发生故障的时候可以在辅助namenode上恢复数据。方法一是就爱那个相关的存储目录复制到新的namenode中。
方法二是使用-importCheckpoint选项启动namenode守护进程，这样会将辅助namenode用作新的主namenode。
借助这个选项，当dfs.name.dir 定义的目录中没有元数据的时候，辅助namenode就从fs.checkpoint.dir中载入最新的检查点数据。否则执行失败。

#### datanode的目录结构

datanode 的存储目录是初始化阶段自动创建的，不需要额外格式化。datanode的关键文件和目录如下：

```
$(dfs.data.dir)/
 |------- current/
           |------- VERSION       //==> version1
           |------- BP-xxxxxx-xxxx-${time}
                     |------ VERSION        //==> version2
                     |------ scanner.cursor
                     |------ tmp/
                     |------ current/
                              |------- dfsUsed
                              |------- finalized/
                              |         |----- subdir0
                              |         |----- subdir1
                              |         .......
                              |
                              |------- rbw/
                                        |----- blk_xxxx_xxxx
                                        |----- blk_xxxx_xxxx.meta

```

其中 version1 处的 VERSION 文件内容如下：

```
#Wed Mar 16 15:34:50 CST 2016
storageID=DS-8538e2ce-2925-4701-9822-b9705f54ec81
clusterID=cluster4
cTime=0
datanodeUuid=0d321f80-e7b3-4cdb-baf2-25d7116964ec
storageType=DATA_NODE
layoutVersion=-56
```

其中 version2 处的 VERSION 文件内容如下：

```
#Wed Mar 16 15:34:50 CST 2016
namespaceID=1598763889
cTime=0
blockpoolID=BP-1745221301-10.16.255.201-1453246819660
layoutVersion=-56
```

内容中的cTime、storageType、layoutVersion的含义跟namenode的VERSION文件中的一样。
其中namespaceID是datanode首次访问namenode时从namenode获取的，而storageID对每个datanode来说都是唯一的。
namenode可以使用这个标识来唯一识别一个datanode。
每次格式化namenode后，clusterID都会变化的，如果datanode的clusterID跟namenode的clusterID不一样，是无法启动集群的。
而datanodeUuid是一个标识码， 不知道有什么用～～～如果您知道，请留言！

blockpoolID跟上面BP-xxxxxx-xxxx-${time}是一样的。

在第二个current目录里的rbt文件都是以blk开头的。这些文件有两种类型：hdfs块文件（原始数据）和块的元数据（meta后缀的）。
块文件包含存储文件的部分原始数据，而元数据文件则包括头部（含版本和类型信息）和这个块个区域的一系列的校验和。

当目录中的数据块数量增加到一定数量的时候， datanode就会创建一个子目录出来存放新的数据块和其元数据信息。
如果当前目录中已经存在64个（由dfs.datanode.numblocks来设置）数据块时，就会创建一个子目录出来存放这些数据块。
其目标就是设计一颗高扇出的目录树。这样做是因为即使有非常多的块，目录树的层数也不会多，方便了datanode管理各个目录中的文件，避免了多数操作系统所遇到的难题。

如果dfs.data.dir指定了不同磁盘上的多个目录，那么数据块会以rr（轮询）的方式写到各个目录中。
这里需要注意的是，即使指定了多个目录，但同一个datanode上的每个磁盘上的块都不会重复，只有不同的datanode上的数据块才可能会重复。

### 安全模式

在namenode启动的时候，namenode会将fsimage载入到内存当中，并执行编辑日志（edist）中的各种操作。
一旦在内存中成功建立文件系统元数据的映像，则创建一个新的fsimage文件和一个空的编辑日志。这个时候，namenode开始监听rpc和http请求。
不过这个时候系统运行在安全模式下，系统对客户端来说是只读的， 而且是只有进行访问文件系统元数据的操作才会成功，
只有当datanode上的块可用时，才能够进行读取块内容的操作。

需要注意的是，系统中的数据块的位置并不是由namenode维护的， 而是由块列表的形式存储在datanode中。
在安全模式下，各个datanode会向namenode发送最新的块列表的信息，在namendoe了解到足够的块位置的时候，系统就可以高效地运行了。
实际上，在安全模式下，namenode不会向datanode发出任何的复制或者删除的指令的。
在系统启动一个刚刚格式化的hdfs集群时，因为系统还没有任何一个块，所以系统这个时候是不会进入安全模式的。

[1]: http://www.spoofer.top/assets/images/2016/03/edist-adn-fsimage.png
