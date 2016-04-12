---
layout: post
title:  "spark-submit 参数"
date:   '2016-04-11 19:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'spark'
excerpt: 'spark-submit'
keywords: 'spark spark-submit 参数'
---

### 概述

本文讲述spark-submit指令的参数及其作用简介。

<!--more-->

##### ---master master_url

这个master_url可以是如下格式：

```
spark://host:port   #standalone
mesos://host:port
yarn
yarn-cluster
yarn-client
local
```
##### --deploy-mode deploy-mode

Driver 程序的运行模式，client或者cluster

##### --class class_name

主类的名称，包括完整的包路径。

##### --name name

spark application的名字。

##### --jars jars

加入到Driver和集群executor的类路径中的jar包列表，以逗号进行分隔。

##### --py-files py-files

使用逗号分隔的放置在python应用程序PYTHONPATH 上的.zip, .egg, .py的文件列表。

##### --files files

使用逗号分隔的每个executor运行时需要的配置文件列表。

##### --properties-file file

设置应用程序属性的文件路径，默认是$SPARK_HOME/conf/spark-defaults.conf。

##### --driver-memory mem

Driver 程序运行时需要的内存， 默认为512M。

##### --driver-java-options

Driver应用程序运行时的一些java配置选项，例如GC的相关信息等。

##### --driver-library-path path

Driver程序依赖的第三方jar包。

##### --executor-memory mem

executor的运行内存大少， 默认为1G。

##### --driver-cores num

Driver的运行核数，默认为1个，仅限于standalone模式。

##### --supervise

失败后是否重启Driver， 仅限与standalone模式。

##### --total-executor-cores num

executor的运行使用的总核数，仅限与standalone和spark on mesos。

##### --executor-cores num

executor运行使用的核数，默认为1， 仅限于spark on yarn模式。

##### --queue queue_name

提交应用程序给哪个yarn队列， 默认的队列是default队列， 仅限与spark on yarn模式。

##### --num-executors num

启动的executor个数。默认为2， 仅限于spark on yarn模式。

##### --archives archives

以逗号分隔的归档文件列表，会被解压到每个executor的工作目录， 仅限于spark on yarn模式。
