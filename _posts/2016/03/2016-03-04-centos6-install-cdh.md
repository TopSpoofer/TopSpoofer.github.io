---
layout: post
title:  "在centos6上安装cm和cdh"
date:   '2016-03-04 14:00:00'
author: 'spoofer'
categories: 'blog'
targets: 'cdh'
excerpt: 'centos6上安装cm和cdh 5.5.1'
keywords: 'centos6,安装cm 5.5.1,安装cdh 5.5.1,大数据'
---

### 概述
本文主要记录如何在centos6上安装clouder-manger和cdh。使用的cdh版本为5.51。下面所有操作都是在root用户登陆下进行的！

### 机器准备
这里模拟一个最小的分布式集群，需要3台centos6（我使用的是centos6.7）。首先配置好每台机器的ip（后面有讲述）， 使其可以远程登陆。
这里介绍一个不错的谷歌浏览器插件：Serverauditor - SSH clien， 这是一个ssh客户端。
#####配置机器ip如下：
```
xxx.xxx.xxx.2   master
xxx.xxx.xxx.4   slave1
xxx.xxx.xxx.6   slave2
```

配置完每台机器ip后，在每台机器的/etc/hosts文件中加入所有机器的ip和域名。如果每台机器都能ping通， 说明这步已经完成了。
