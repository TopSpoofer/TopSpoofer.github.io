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

1、因为kafka的分布式安装需要依赖于zookeeper，所以需要先安装分布式的zookeeper。我使用的是cdh-5.5.1的zookeeper集群（不要问我为什么不是要cdh的kafka集群，为时所迫啊）。
2、下载kafka二进制安装包。 下载地址： (download kafka)[http://kafka.apache.org/downloads.html]
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
$ cp config/server.properties etc/kafka-server2.properties  ## Kafka2机器上不需要执行这个，因为已经够3个block了。

### 假加入环境变量
$ echo ### kafka >> ~/.bashrc
$ echo export KAFKA_HOME=/home/spoofer/Kafka >> ~/.bashrc ### /home/spoofer/Kafka为你的kafka的安装目录
$ echo export KAFKA_BIN=$KAFKA_HOME/bin >> ~/.bashrc
$ echo export PATH=$KAFKA_BIN:$PATH
$ source ~/.bashrc
```
经过上述步骤，kafka已经安装好了，但还不能使用，需要进行配置。

#### 配置kafka
