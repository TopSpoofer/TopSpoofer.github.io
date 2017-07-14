---
layout: post
title:  "基于Nested Sets的树形数据库详细设计"
date:   '2017-07-14 09:00:00'
author: 'spoofer'
categories: 'blog'
tags: '数据库'
excerpt: '数据库 mysql 树形表 Nested Sets'
keywords: '数据库 mysql 树形表 Nested Sets'
---


## 概述

前面写了一篇 树形结构的数据库设计文章[Title](http://www.spoofer.top/2017/07/13/%E6%A0%91%E5%BD%A2%E7%BB%93%E6%9E%84%E7%9A%84%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%BB%E7%BB%93)，里面就简单介绍了 Nested Sets 的设计，现在来对Nested Sets 进行深入一点的了解。

一般基于数据库的应用中，查询数据的需求总是要大于修改和删除，为了避免对树进行查询时产生“递归”的过程，
Nested Sets 使用了可以无限分组的左右值编码方案，这样的方案可以无需进行递归查询，更适合无法预料深度的树需求。
<!--more-->


### Nested Sets 的表设计

Nested Sets 需要left和right来记录一个节点的左右值，这个节点左右值是按树的前序遍历来进行编排的。
