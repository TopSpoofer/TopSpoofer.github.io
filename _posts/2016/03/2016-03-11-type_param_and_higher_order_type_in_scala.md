---
layout: post
title:  "scala类型参数和高阶类型简介"
date:   '2016-03-11 14:00:00'
author: 'spoofer'
categories: 'blog'
tags: 'scala'
excerpt: 'scala类型参数和高阶类型简介'
keywords: 'scala 类型参数 高阶类型'
---

### 概述

本文主要简单介绍scala的类型参数和高阶类型。

<!--more-->

### 类型参数

类型参数是在调用方法、构造类型或者扩展类型是作为参数传入的一个类型定义。

```
class People[A](name: String) {
}

class Test {
  def test[A](name: String, a: A) = {
    println(s"$name, $a")
  }
}
```

如上代码， 其中A就是类型参数了。即使是初学者，其实很多时候都用到， 如:
```
val list = List[String]("1", "2")
String就是这个list的类型参数了

```

### 高阶类型

高阶类型函数是接受其他函数为参数的函数一样，高阶类型则是接受其他类型作为参数构造出来新类型的类型。
高阶类型可以有一个或者多个其他类型作为参数。在scala里可以用type关键字来构造高阶类型。

![higher_order_type.png][1]

如上图，这里定义了一个叫做callback的高阶函数类型。callback类型接受一个类型参数，
创建一个新的function类型，在参数化之前，callback不是一个完整类型。
这里定义的callback可以用来简化那种接受一个参数且无返回值的函数的标签名字。

除了用来简化复杂类型的类型签名外，高阶类型也用于使复杂类型符合想调用的方法所要求的简单类型签名。如下：

![higher_order_type2.png][2]

foo接受一个类型M ---- M是一个未知的参数化类型。_关键字是一个占位符，用来指代一个未知的存在类型。
callback作为参数传给foo方法，并用前面定义的x作为方法的参数调用之。

[1]: http://www.spoofer.top/assets/images/2016/03/higher_order_type.png
[2]: http://www.spoofer.top/assets/images/2016/03/higher_order_type2.png
