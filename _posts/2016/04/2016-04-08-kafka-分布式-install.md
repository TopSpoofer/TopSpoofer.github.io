---
layout: post
title:  "kafka 0.9.0.1 分布式集群安装"
date:   '2016-04-08 07:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'spark'
excerpt: 'kafka 0.9.0.1 分布式集群安装'
keywords: 'kafka 0.9.0.1 分布式 安装'
---

### 概述

本文主要讲述kafka分布式的安装。其实网络上已经有很多这方面的资料，这篇文章主要是记录我自己学习的过程！

<!--more-->

#### 安装前准备

1、因为kafka的分布式安装需要依赖于zookeeper，所以需要先安装分布式的zookeeper。我使用的是cdh-5.5.1的zookeeper集群（不要问我为什么不是要cdh的kafka集群，为势所迫啊）。

2、下载kafka二进制安装包。 下载地址：   (download kafka)[http://kafka.apache.org/downloads.html]

3、准备集群机器： kafka1、kafka2（条件限制只准备了2台机器，我的业务也不需要太大的集群）

4、对机器安装ssh，测试每台集群都能免密码登陆，包括zookeeper的机器哦。

5、对集群机器安装java。


#### 安装kafka

对kafka1和Kafka2进行以下操作：
将kafka安装在/home/$user的家目录，假如user是spoofer。

```
$ tar xvf kafka_2.10-0.9.0.1.tgz
$ mv kafka_2.10-0.9.0.1 Kafka
$ cd Kafka

## 创建保存log的目录，因为是启动3个block服务，这里创建了3个目录。
$ mkdir data
$ mkdir data/blk1
$ mkdir data/blk2
$ mkdir data/blk3

$ mkdir etc
$ cp config/server.properties etc/kafka-server1.properties
$ cp config/server.properties etc/kafka-server2.properties  ## Kafka2机器上不需要执行这个，因为已经够3个broker了。

### 加入kafka home到环境变量
$ echo ### kafka >> ~/.bashrc
$ echo export KAFKA_HOME=/home/spoofer/Kafka >> ~/.bashrc ### /home/spoofer/Kafka为你的kafka的安装目录
$ echo export KAFKA_BIN=$KAFKA_HOME/bin >> ~/.bashrc
$ echo export PATH=$KAFKA_BIN:$PATH
$ source ~/.bashrc
```
经过上述步骤，kafka已经安装好了，但还不能使用，需要进行配置。

#### 配置kafka

因为只是简单的安装，所以配置都是简单的配置，没有调优等。

主要需要修改的几个配置项为：

```
broker.id # 每个broker服务都要不同，而且大于0
listeners=PLAINTEXT://:9092 # broker监听的端口
port=9092
host.name=Kafka1
advertised.host.name=Kafka1
advertised.port=9092
log.dirs=/home/spoofer/Kafka/data/blk1 # 日志数据保存的目录
zookeeper.connect=Core1:2181,Core2:2181,Core3:2181 # zookeeper集群
delete.topic.enable=true
```
我的集群架构是： 在Kafka1上安装两个broker，在Kafka2上安装一个broker。对于上述的几个配置项，我集群机器的详细配置如下：

```
### kafka1， broker1， kafka-server1.properties
broker.id=1
listeners=PLAINTEXT://:9092
port=9092
host.name=Kafka1
advertised.host.name=Kafka1
advertised.port=9092
log.dirs=/home/spoofer/Kafka/data/blk1
zookeeper.connect=Core1:2181,Core2:2181,Core3:2181
delete.topic.enable=true



### kafka1， broker2， kafka-server2.properties
broker.id=2
listeners=PLAINTEXT://:9093
port=9093
host.name=Kafka1
advertised.host.name=Kafka1
advertised.port=9093
log.dirs=/home/spoofer/Kafka/data/blk2
zookeeper.connect=Core1:2181,Core2:2181,Core3:2181
delete.topic.enable=true



### kafka2， broker1， kafka-server1.properties
broker.id=3
listeners=PLAINTEXT://:9094
port=9094
host.name=Kafka2
advertised.host.name=Kafka2
advertised.port=9094
log.dirs=/home/hadoop/DMP/Kafka/data/blk3
zookeeper.connect=Core1:2181,Core2:2181,Core3:2181
delete.topic.enable=true
```
除了这些，还需要到/home/spoofer/Kafka/config/里编辑 producer.properties 文件，要修改的配置项为：

```
### kafka集群里每一台机器都需要修改， 根据需要进行配置
metadata.broker.list=Kafka1:9092,Kafka1:9093,Kafka2:9094
```

到这里，kafka集群的安装和配置都已经好了，下面进行测试。


### 使用kafka

#### 启动kafka

这个是启动broker的指令，其他broker自己设置指定的配置文件进行启动

```
bin/zookeeper-server-start.sh etc/kafka-server1.properties
```

或者以守护进程来运行

```
bin/zookeeper-server-start.sh -daemon etc/kafka-server1.properties
```

#### 创建topic

创建一个叫做"mytest"的topic，它只有1个分区，3个副本。

```
bin/kafka-topics.sh --create --zookeeper Core1:2181 --replication-factor 3 --partitions 1 --topic mytest
```

#### 查看所有topics

```
bin/kafka-topics.sh --list --zookeeper Core:2181
```

#### 查看topic在每个节点的信息

```
bin/kafka-topics.sh --describe --zookeeper Core1:2181 --topic mytest

### 描述如下：
[hadoop@Kafka2 Kafka]$ bin/kafka-topics.sh --describe --zookeeper Core1:2181 --topic mytest
Topic:mytest     PartitionCount:1        ReplicationFactor:3     Configs:
        Topic: mytest    Partition: 0    Leader: 2       Replicas: 2,1,3 Isr: 2,1,3
```

其中字段的解析如下：

leader：负责处理消息的读和写，leader是从所有节点中随机选择的。

replicas：列出了所有的副本节点，不管节点是否在服务中。

isr：是正在服务中的节点。


#### 启动生产者

```
bin/kafka-console-producer.sh --broker-list Kafka2:9094 --topic mytest
```

启动消费者后，键入你的消息，然后回车就可以发送了。


#### 启动消费者

```
bin/kafka-console-consumer.sh --zookeeper Core1:2181 --from-beginning --topic mytest

```

如果想只接受消费者启动后的消息的话，把--from-beginning去掉即可。


#### 删除topic

删除topic需要在broker配置上配置：delete.topic.enable=true

```
kafka-topics.sh --delete --zookeeper Core1:2181 --topic mytest
```

删除kafka存储目录（server.properties文件log.dirs配置，默认为"/tmp/kafka-logs"）相关topic目录
删除zookeeper "/brokers/topics/"目录下相关topic节点



  
