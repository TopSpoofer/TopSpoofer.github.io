---
layout: post
title:  "intellij远程提交任务到spark集群"
date:   '2016-03-16 14:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'intellij'
excerpt: 'intellij远程提交任务到spark集群'
keywords: 'intellij 远程提交任务 spark集群'
---

### 概述

本文介绍如何使用 intellij idea 远程提交任务到配置standnode 的spark集群上运行。

<!--more-->

### 提交任务抛出异常

在运行spark程序时候抛出找不到主类的异常，这个时候是因为本地提交的任务没有将main函数的jar丢到workers上运行。
所以在workers上是找不到你的程序的！

### 配置intellij project 属性

#### 配置代码

新建 sbt project， 这个就不多说了。代码如下：

```
package top.spoofer.hbrdd

import org.apache.spark.{SparkContext, SparkConf}

object HbMain {
  private val master = "Core1"
  private val port = "7077"
  private val appName = "hbase-rdd_spark"
  private val data = "hdfs://Master1:8020/test/spark/hbase/testhb"

  def main(args: Array[String]) {
    val sparkConf = new SparkConf()
      .setMaster(s"spark://$master:$port")
      .setAppName(appName).setJars(List("/home/lele/coding/hbrdd/out/artifacts/hbrdd_jar/hbrdd.jar"))

    val sc = new SparkContext(sparkConf)
    val ret = sc.textFile(data).map({ line =>
      val Array(k, col1, col2, _) = line split "\t"
      val content = Map("col1" -> col1, "col2" -> col2)
      k -> content
    })

    println(ret.count())

    sc.stop()
  }
}
```

工程包结构如下图：

![project-code.png][1]

刚才说了是因为我们的工程jar包没有被提交到worker上，所以我们可以手动将我们的jar加入到要提交jar list里去。
使用SparkConf的setJars加入jar， 代码如下：

```
//其中jarPath是你打包出来的jar，打包操作下面有述
val jarPath = "/home/lele/coding/hbrdd/out/artifacts/hbrdd_jar/hbrdd.jar"
val sparkConf = new SparkConf().setMaster(s"spark://$master:$port").setAppName(appName)
.setJars(List(jarPath))

```
这样提交job的时候我们的jar就一起被提交了。

现在代码配置好了，需要配置 intellij，使其帮我们把项目打包成jar。具体步骤如下：

#### 创建Artifacts

点击：File->project structure->Artifacts，添加一个Artifacts， 如下图：

![new-artif.png][2]

#### 配置创建好的Artifacts

不多说，看下图：

![create-artif.png][3]

#### 去除已有的依赖

因为集群上已经有scala的lib和spark assembly的lib，所以这里去除这些依赖。如果没有就不用了。
如果把这些依赖都打包成jar， 那么这个jar会非常大！因为spark assembly就有100多M。

如图：

![ref-remove.png][4]

到此，应该可以正确地运行程序了。

[1]: http://www.spoofer.top/assets/images/2016/03/project-code.png
[2]: http://www.spoofer.top/assets/images/2016/03/new-artif.png
[3]: http://www.spoofer.top/assets/images/2016/03/create-artif.png
[4]: http://www.spoofer.top/assets/images/2016/03/ref-remove.png
