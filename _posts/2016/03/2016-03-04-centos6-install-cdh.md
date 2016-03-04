---
layout: post
title:  "在centos6上安装cm和cdh"
date:   '2016-03-04 14:00:00'
author: spoofer
categories: blog
target: cdh
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

### 配置每台机器的ip和hostname，修改dns配置
##### 修改主机名字
```
vi /etc/sysconfig/network
HOSTNAME=master  #修改localhost.localdomain为master
```
##### 修改每台机器的ip
这里我们为每台机器配置静态ip，使用ifconfig -a查看你系统所有的网卡，下面我们编辑eth0这张网卡。
```
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
配置内容为（每台机器的ip不相同！）：
```
DEVICE=eth0 #描述网卡对应的设备别名，例如ifcfg-eth0的文件中它为eth0
BOOTPROTO=static #设置网卡获得ip地址的方式，可能的选项为static，dhcp或bootp，分别对应静态指定的 ip地址，通过dhcp协议获得的ip地址，通过bootp协议获得的ip地址
BROADCAST=192.168.0.255 #对应的子网广播地址
HWADDR=00:07:E9:05:E8:B4 #对应的网卡物理地址，虚拟机下我没有配置这个也没有问题
IPADDR=192.168.0.1 #如果设置网卡获得 ip地址的方式为静态指定，此字段就指定了网卡对应的ip地址
NETMASK=255.255.255.0 #网卡对应的网络掩码
NETWORK=192.168.0.0 #网卡对应的网络地址
ONBOOT=yes
GATEWAY=192.168.0.1

```

##### 修改dns配置
```
vi /etc/resolv.conf
```
配置内容：
```
nameserver    114.114.114.114
```

##### 重启服务
```
service network restart 　或 　 /etc/init.d/network restart
```
这样你的系统应该上网了，ping一下百度试一下。到这里集群的每台机器可以上网，互相ping通了。


### 关闭防火墙
##### 关闭iptables
```
service iptables stop
chkconfig iptables off
```
##### 关闭selinux
```
vi /etc/selinux/config
```
配置内容为：
```
设置或者加入：SELINUX=disabled
```

### 安装ntp（时间服务器）
这个ntp是为了同步集群的时间的，集群很多组件都依赖集群机器的时间同步，如hadoop这个组件在集群时间不同步是会出现一些莫名其妙的问题，所以这一步还是非常必要的。

##### 安装ntp
```
yum install -y ntp
```
##### 设置开机启动
```
chkconfig ntpd on
```

### 配置免密码登陆
安装ssh server
```
yum install -y openssh-server
```
