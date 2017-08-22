---
layout: post
title:  "docker-compose mysql5.7 主从部署"
date:   '2017-08-22 16:00:00'
author: 'spoofer'
categories: '笔记'
tags: '数据库'
excerpt: 'docker docker-compose mysql 5.7 集群 主从'
keywords: 'docker docker-compose mysql 5.7 集群 主从'
---

## 概述
本文记录了利用docker compose部署mysql 5.7 主从节点的踩坑过程...

安装docker mysql的过程请参看[docker-compose 安装mysql5.7](http://www.spoofer.top/2017/08/07/docker-compose%E5%AE%89%E8%A3%85mysql5.7)

<!--more-->

### 准备
本次将在ubuntu 16.04上部署一个master 节点和一个slave节点。
进入工作路径（随便一个目录即可），这里是：
```
cd /home/ubuntu/mysql/cluster_test/
```

创建master和slave目录：
```
mkdir master
mkdir slave

#分别在 master 和 slave中创建data目录
cd master
mkdir data
cd ../slave
mkdir data
```

编写 master 的 mymysqld.cnf（进入到master目录中）内容为:
```
# Here is entries for some specific programs
# The following values assume you have at least 32M ram

[mysqld_safe]
socket        = /var/run/mysqld/mysqld.sock
nice        = 0

[mysqld]
#
# * Basic Settings
#
user        = mysql
pid-file    = /var/run/mysqld/mysqld.pid
socket        = /var/run/mysqld/mysqld.sock
port        = 3306
basedir        = /usr
datadir        = /var/lib/mysql
tmpdir        = /tmp
lc-messages-dir    = /usr/share/mysql
skip-external-locking
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address        = 0.0.0.0
#
# * Fine Tuning
#
key_buffer_size        = 16M
max_allowed_packet    = 16M
thread_stack        = 192K
thread_cache_size       = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam-recover-options  = BACKUP
#max_connections        = 100
#table_cache            = 64
#thread_concurrency     = 10
#
# * Query Cache Configuration
#
query_cache_limit    = 1M
query_cache_size        = 16M
#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file        = /var/log/mysql/mysql.log
#general_log             = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Here you can see queries with especially long duration
#log_slow_queries    = /var/log/mysql/mysql-slow.log
#long_query_time = 2
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
server-id        = 1
log_bin            = /var/log/mysql/mysql-bin.log
expire_logs_days    = 10
max_binlog_size   = 100M
#binlog_do_db        = include_database_name
#binlog_ignore_db    = include_database_name
#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!
#
# * Security Features
#
# Read the manual, too, if you want chroot!
# chroot = /var/lib/mysql/
#
# For generating SSL certificates I recommend the OpenSSL GUI "tinyca".
#
# ssl-ca=/etc/mysql/cacert.pem
# ssl-cert=/etc/mysql/server-cert.pem
# ssl-key=/etc/mysql/server-key.pem
```
需要注意的是 master 配置中 server-id = 1，这个是唯一id来的，slave中配置跟master一样，只需要改变server-id为2。
除了server-id需要设置外，还需要设置log_bin（必须启用二进制日志），这里默认即可。

### 编写Dockerfile
在 master 目录中创建Dockerfile文件，内容为：
```
FROM mysql:5.7

COPY mymysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf

EXPOSE 3306

CMD ["mysqld"]
```
同样在slave中进行相同的操作。

### 构建docker 镜像

```
cd master
#需要root权限
docker build -t mysql-master .
```
在slave也需要同样的操作：
```
cd slave
docker build -t mysql-slave .
```

### 编写 docker-compose.yml 文件

在工作目录新建docker-compose.yml， 内容为：

```
version: '3'
services:
  mysql_master_compose:
    image: mysql-master
    container_name: mysql_master_compose
    ports:
      - 7706:3306
    volumes:
      - /home/ubuntu/mysql/cluster_test/master/data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: "123456"

  mysql_slave_compose:
    image: mysql-slave
    container_name: mysql_slave_compose
    ports:
      - 8806:3306
    volumes:
      - /home/ubuntu/mysql/cluster_test/slave/data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
```

### 运行mysql实例

此时可以在做过目录运行两个mysql的实例了：
```
docker-compose up
#后台运行 docker-compose up -d
```

### 设置主从复制

#### 配置master

进入master的命令行并且登录mysql：
```
docker exec -it mysql-master bash
mysql -u root -p
#输入 MYSQL_ROOT_PASSWORD 设置的密码值进行登录
```
创建主从复制的用户：
```
mysql> create user spoofer;
```

给用户授权：
```
mysql> GRANT REPLICATION SLAVE ON *.* TO 'spoofer'@'%' IDENTIFIED BY '123456';
#这里权限有点大，需要自己进行调整
```

进行锁表操作：

```
FLUSH TABLES WITH READ LOCK;
#锁库,不让数据再进行写入动作,这个命令在结束终端会话的时候会自动解锁
```

查看master状态：
```
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      991 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

需要记下： mysql-bin.000003 和 991这两个字段，在配置slave 时需要用到。

#### 配置slave
进入master的命令行并且登录mysql：
```
docker exec -it mysql-slave bash
mysql -u root -p
#输入 MYSQL_ROOT_PASSWORD 设置的密码值进行登录
```

链接master：
```
mysql> change master to master_host='mysql_master_compose',master_port=3306,master_user='spoofer',master_password='123456',master_log_file='mysql-bin.000003',master_log_pos=991;
```
可以看到host用的是node的名字的，端口是3306，因为docker-compose在内部会维持一个网络，所以节点间可以直连。

启动slave：
```
mysql> start slave;
```

现在在mysql-master实例上创建一个数据库，看看mysql-slave实例上是不是同步了？
还有一点就是，我在mysql-slave上修改了database的索引，主库是没有更新的。
