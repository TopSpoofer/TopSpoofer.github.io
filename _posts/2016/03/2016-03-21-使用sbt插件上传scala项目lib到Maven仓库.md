---
layout: post
title:  "使用sbt插件上传scala项目打包成的lib到Maven仓库"
date:   '2016-03-21 14:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'sbt'
excerpt: '使用sbt插件上传scala项目打包成的lib到Maven仓库'
keywords: 'sbt 插件 上传scala项目 library Maven仓库'
---

### 概述

由于没有比较好的中文教程讲解如何将scala项目打包成lib并且上传到Maven仓库，即使是sbt的官网的英文教程也是不是很详细，
所以本文主要讲解如何使用sbt插件将您的scala项目上传到Maven仓库里。

 <!--more-->

### 创建sbt项目

这里要创建一个sbt项目并且push到github上。

我的项目结构如下：

![jslog-sbt-project.png][1]

需要注意的是，我的项目结构里的scala->top->spoofer。这个是下面有讲的gruopID！


### 上传前准备

#### 注册 sonatype 的账号

很容易理解， 如果要上传你的项目的mvn， 那么你必须要有一个身份！到 https://issues.sonatype.org/login.jsp 注册账号。
然后创建一个issue如下图：

![create-issue-1.png][2]

![create-issue-2.png][3]

你需要创建的是一个“newproject”类型的issue。其中project是你项目在github的地址。
而scm url是你项目的地址，即用git工具来提交代码时用的那个地址，不过这里不支持ssh类型的地址，所以只能使用https的。
特别要注意的是groupID这个项，我实践的时候一直不成功就是因为这个！我的域名是www.spoofer.top, 所以我选择使用了top.spoofer这个id。
这个需要和上面的sbt项目提到的对应！

创建issue后， 需要等待的，时间长短就不知道了,可能需要1~2天来。如果审核通过，则会收到审核通过的邮件。如下：

![issue-configuration.png][4]

收到了审核通过的邮件后， 我们可以进行下一步了。

### 安装sbt插件

在项目的project/ 目录下有个plugins.sbt文件， 内容如下：

```
logLevel := Level.Warn

addSbtPlugin("org.xerial.sbt" % "sbt-sonatype" % "1.1")

addSbtPlugin("com.jsuereth" % "sbt-pgp" % "1.0.0") // fot sbt-0.13.5 or higher
```

需要安装sbt-sonatype和sbt-pgp插件。
其中sbt-sonatype插件可以帮助上传jar到mvn，而sbt-pgp可以生成发布时需要的key。


### 编写build.sbt和sonatype.sbt


在sbt中加入如下内容：

```
publishTo := {
  val nexus = "https://oss.sonatype.org/"
  if (isSnapshot.value)
    Some("snapshots" at nexus + "content/repositories/snapshots")
  else
    Some("releases"  at nexus + "service/local/staging/deploy/maven2/")
}
```

新建sonatype.sbt， 在sonatype.sbt中的内容如下：

```
organization := "top.spoofer"

publishMavenStyle := true

publishArtifact in Test := false

pomIncludeRepository := { _ => false }

pomExtra in Global := {
    <url>https://github.com/TopSpoofer/jslog</url>
    <licenses>
      <license>
        <name>Apache 2</name>
        <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
      </license>
    </licenses>
    <scm>
      <url>git@github.com:TopSpoofer/jslog.git</url>
      <connection>scm:git:git@github.com:TopSpoofer/jslog.git</connection>
    </scm>
    <developers>
      <developer>
        <id>TopSpoofer</id>
        <name>spoofer</name>
        <url>https://github.com/TopSpoofer/</url>
      </developer>
    </developers>
}
```
其中organization是比较重要的，这是跟groupID相关的。

这下项目的配置已经好了。

### 发布项目

加入项目根目录， 终端执行sbt。

```
> compile          //编译项目
> pgp-cmd gen-key  //生成key
> pgp-cmd send-key keyname hkp://pool.sks-keyservers.net //将公钥发到服务器，其中keyname是生成key时使用的用户名字或者邮箱。
```

如果在send-key过程中，没有任何输出，那么是你的keyname错了，重新确认一下。

新建文件：~/.sbt/0.13/sonatype.sbt，在其中加入：

```
credentials += Credentials("Sonatype Nexus Repository Manager",
                           "oss.sonatype.org",
                           "<your username>",
                           "<your password>")
```

发布：

```
> publishSigned
[info] Packaging /home/lele/coding/jslog/target/scala-2.10/jslog_2.10-1.0-javadoc.jar ...
[info] Packaging /home/lele/coding/jslog/target/scala-2.10/jslog_2.10-1.0-sources.jar ...
[info] Updating {file:/home/lele/coding/jslog/}jslog...
[info] Done packaging.
[info] Done packaging.
[info] Wrote /home/lele/coding/jslog/target/scala-2.10/jslog_2.10-1.0.pom
[info] Resolving org.fusesource.jansi#jansi;1.4 ...
[info] Done updating.
[info] :: delivering :: top.spoofer#jslog_2.10;1.0 :: 1.0 :: release :: Mon Mar 21 16:15:13 CST 2016
[info] 	delivering ivy file to /home/lele/coding/jslog/target/scala-2.10/ivy-1.0.xml
[info] Compiling 5 Scala sources to /home/lele/coding/jslog/target/scala-2.10/classes...
[info] Packaging /home/lele/coding/jslog/target/scala-2.10/jslog_2.10-1.0.jar ...
[info] Done packaging.
Please enter PGP passphrase (or ENTER to abort): *********
[info] 	published jslog_2.10 to https://oss.sonatype.org/service/local/staging/deploy/maven2/top/spoofer/jslog_2.10/1.0/jslog_2.10-1.0.pom.asc
[info] 	published jslog_2.10 to https://oss.sonatype.org/service/local/staging/deploy/maven2/top/spoofer/jslog_2.10/1.0/jslog_2.10-1.0-javadoc.jar.asc
[info] 	published jslog_2.10 to https://oss.sonatype.org/service/local/staging/deploy/maven2/top/spoofer/jslog_2.10/1.0/jslog_2.10-1.0.jar.asc
[info] 	published jslog_2.10 to https://oss.sonatype.org/service/local/staging/deploy/maven2/top/spoofer/jslog_2.10/1.0/jslog_2.10-1.0.pom
[info] 	published jslog_2.10 to https://oss.sonatype.org/service/local/staging/deploy/maven2/top/spoofer/jslog_2.10/1.0/jslog_2.10-1.0-sources.jar.asc
[info] 	published jslog_2.10 to https://oss.sonatype.org/service/local/staging/deploy/maven2/top/spoofer/jslog_2.10/1.0/jslog_2.10-1.0.jar
[info] 	published jslog_2.10 to https://oss.sonatype.org/service/local/staging/deploy/maven2/top/spoofer/jslog_2.10/1.0/jslog_2.10-1.0-javadoc.jar
[info] 	published jslog_2.10 to https://oss.sonatype.org/service/local/staging/deploy/maven2/top/spoofer/jslog_2.10/1.0/jslog_2.10-1.0-sources.jar
[success] Total time: 43 s, completed 2016-3-21 16:15:55
```

到这里，你的lib已经上传到https://oss.sonatype.org/#welcome里了。注册登陆后点击左边的Staging Repositories选项， 如下：


![jslog-upload.png][5]

![jslog-jar.png][6]

注意，这时项目的状态是open的，现在检测你刚才publishSigned上传的信息是否齐全，如果你操作正确，
Repository字段对应的是类似与groupID然后加划线加序号，如下：

```
topspoofer-1000
```

这时点击勾选上传的Repository， 然后点击close。然后再回去创建issue的地方， 添加一条评论：
```
This is the first time I promoted a release. Could you activate the sync process please?
```

等待几个小时后会收到工作人员的邮件，然后会发现原来的 Release 按钮变成了可用状态，点击后就完成了发布操作。
接着在回到 Issue 处添加以下评论等待几个小时就可以在 Maven 中央仓库（http://mvnrepository.com/artifact/top.spoofer/jslog_2.10/1.0）看到你的劳动成果了。实际上我等了大约2日，仓库就同步了。

[1]: http://www.spoofer.top/assets/images/2016/03/jslog-sbt-project.png
[2]: http://www.spoofer.top/assets/images/2016/03/create-issue-1.png
[3]: http://www.spoofer.top/assets/images/2016/03/create-issue-2.png
[4]: http://www.spoofer.top/assets/images/2016/03/issue-configuration.png
[5]: http://www.spoofer.top/assets/images/2016/03/jslog-upload.png
[6]: http://www.spoofer.top/assets/images/2016/03/jslog-jar.png
