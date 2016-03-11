---
layout: post
title:  "scala类型系统中dot(.)符号与hash(#)符号的主要区别"
date:   '2016-03-011 14:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'scala'
excerpt: 'scala 类型系统 dot(.)符号与hash(#)符号'
keywords: 'scala 类型系统 dot(.)符号与hash(#)符号'
---

### 概述

本文主要讲述在scala语言的类型系统中, dot（.）符号与hash（#）符号的主要区别。

scala version 2.10.5

<!--more-->

### code

```
class Outer {
    trait Inner
    def y = new Inner
    def foo(x: this.Inner) = null
    def bar(x: x#Inner) = null
}

val x = nwe Outer
﻿val y = new Outer

```

如上代码，有两种机制来应用scala定义的类型：hash（#）和dot（.）操作符。对类型使用dot操作符可以想成与对象的成员用dot操作符有一样的效果。它指向某对象实例里找到的类型，称为路径依赖类型。而hash操作符比dot操作符要宽松。它被称作类型注入。类型注入是一种引用嵌套类型而有不需要对象实例化路径的方式。

当使用x.foo(x.y)时，返回的是null，但使用x.foo(y.y)时会编译不过。因为this.Inner是实例对象的Inner。如果使用x.bar(y.y)的话会返回null，因为这里路径赖类型为任何的foo实例里的bar实例。
