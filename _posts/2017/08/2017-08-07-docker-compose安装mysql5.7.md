---
layout: post
title:  "docker-compose 安装mysql5.7"
date:   '2017-08-07 16:00:00'
author: 'spoofer'
categories: '笔记'
tags: '数据库'
excerpt: 'docker docker-compose mysql 5.7'
keywords: 'docker docker-compose mysql 5.7'
---

## 概述

本文简单地使用docker 安装mysql 5.7，并使用docker-compose进行部署。

<!--more-->

### docker中安装mysql 5.7镜像

国外的镜像下载比较慢，这里使用的阿里的镜像。


```
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -

```

可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器：

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ejburpg5.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

拉取镜像：
```
docker pull mysql:5.7

docker images | grep mysql # 查看镜像是否已经安装了

```

#### 配置docker-compose 文件

创建需要的目录：

```
mkdir -p /home/ubuntu/mysql/data
mkdir -p /home/ubuntu/mysql/conf
```

需要注意的是，这两个目录你需要自己定，不一定跟这里一样～

新建目录后，将你需要的mysql配置文件 mymysqld.cnf 放到conf目录里：

```
#
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
#
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

# This will be passed to all mysql clients
# It has been reported that passwords should be enclosed with ticks/quotes
# escpecially if they contain "#" chars...
# Remember to edit /etc/mysql/debian.cnf when changing the socket location.

# Here is entries for some specific programs
# The following values assume you have at least 32M ram

[mysqld_safe]
socket		= /var/run/mysqld/mysqld.sock
nice		= 0

[mysqld]
#
# * Basic Settings
#
user		= mysql
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
port		= 3306
basedir		= /usr
datadir		= /var/lib/mysql
tmpdir		= /tmp
lc-messages-dir	= /usr/share/mysql
skip-external-locking
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address		= 0.0.0.0
#
# * Fine Tuning
#
key_buffer_size		= 16M
max_allowed_packet	= 16M
thread_stack		= 192K
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
query_cache_limit	= 1M
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
#log_slow_queries	= /var/log/mysql/mysql-slow.log
#long_query_time = 2
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
#server-id		= 1
#log_bin			= /var/log/mysql/mysql-bin.log
expire_logs_days	= 10
max_binlog_size   = 100M
#binlog_do_db		= include_database_name
#binlog_ignore_db	= include_database_name
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

上面的配置是默认的配置， 我只改了 bind-address		= 0.0.0.0， 用于测试时可以远程链接。


编写docker-compose 文件docker-compose.yml:

```
version: '3'
services:
  mysql_compose:
    image: mysql:5.7
    container_name: mysql_compose
    ports:
      - 6606:3306
    volumes:
      - /home/ubuntu/mysql/data:/var/lib/mysql
      - /home/ubuntu/mysql/conf/mymysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
```
Docker使用-p选项允许容器上的端口映射到主机上的端口。如果你如上所述启动容器（6606:3306），你可以通过将客户机连接到主机上的端口(6606)来连接到数据库。

### 运行mysql image

进入到上面编写的docker-compose.yml文件的目录，运行：

```
docker-compose up -d
```
