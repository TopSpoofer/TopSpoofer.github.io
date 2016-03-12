---
layout: post
title:  "scala高阶函数简介"
date:   '2016-03-12 09:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'scala'
excerpt: 'scala 高阶函数'
keywords: 'scala 高阶函数'
---

### 概述

本文主要是简单讲解scala中高阶函数形式。

<!--more-->

### 匿名函数

```
(x: Int) => 3 * x
```
如上函数，将参数x乘以3。

可以将这个匿名函数放在变量里：

```
val tmp = (x: Int) => 3 * x
val ret = tmp(3)  //结果为9
```
也可以将其传递给另一个函数：

```
val ret = Array(1, 2, 3).map(tmp) //Array(3, 6, 9)
```
更可以使用像map { tmp } 这样的用法，在使用中置表示法时（没有句点），这样的写法比较常见。如下：

```
val ret = Array(1, 2, 3) map { tmp }
```

### 带函数参数的函数

```
def test(f: (Int) => Int) = f(10)
```
如上代码， 函数test接受一个函数参数并且返回Int，而这个函数参数的形式是：接受一个Int参数并且返回Int。
可以这样使用：

```
val ret= test(tmp)  //30
```
所以函数test的类型为：( (Int) => Int ) => Int

高阶函数也可以产出另一个函数，如下：

```
def  mulBy(factor: Int) = (x: Int) => factor * x
val mul = mulBy(5)
val ret = mul(100)  //500
```

可以看出，mulBy接受一个Int的参数，返回一个类型为(Int) => Int 的函数，
所以其类型为： (Int) => ( (Int) => Int )
