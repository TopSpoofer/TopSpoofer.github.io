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

前面写了一篇 [树形结构的数据库设计](http://www.spoofer.top/2017/07/13/%E6%A0%91%E5%BD%A2%E7%BB%93%E6%9E%84%E7%9A%84%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%BB%E7%BB%93)文章，里面就简单介绍了 Nested Sets 的设计，现在来对Nested Sets 进行深入一点的了解。

一般基于数据库的应用中，查询数据的需求总是要大于修改和删除，为了避免对树进行查询时产生“递归”的过程，
Nested Sets 使用了可以无限分组的左右值编码方案，这样的方案可以无需进行递归查询，更适合无法预料深度的树需求。
<!--more-->


### Nested Sets 的表设计
Nested Sets 需要left和right来记录一个节点的左右值，这个节点左右值是按树的前序遍历来进行编排的。

| id        | left  | right | name |
| ------------- |:-------------| :---------|:--------|
| 节点id      | 节点左值 | 节点右值 | 业务字段 |

需要加入其他字段来完成业务的需求。

### Nested Sets 的left 和right 编排
前面提到过，left和right是按树的前序遍历来进行编排的，那么先来复习一下树的前序遍历，如下边的树图：

![NestedSets.png][1]

前序遍历就是父节点优先遍历然后再到左树节点再到右树节点，所以遍历循序就是图中橙色先的方向和循序。
所以对于这棵简单的树，前序遍历的循序就是 boss -> c# leader -> c# programmer -> java leader -> java programmer。

所以我们可以得到下表：

| id        | left  | right | name |
| ------------- |:-------------| :---------|:--------|
| 1      | 1 | 10 | boss |
| 2      | 2 | 5 | C# leader |
| 3      | 6 | 9 | Java leader |
| 4      | 3 | 4 | C# programmer |
| 5      | 7 | 8 | java programmer |

### Nested Sets 的curd

要完成业务，我们必须要能够对树进行crud操作，需要构造出与之对应的相关算法。
我们将表命名为 tree。

#### 获取某节点的子孙节点

获取某节点的子孙节点这个是最基本的需求了。

以boss为例，只需要简单的分析就可以知道，left > 1 right < 10的节点都是boss的小弟！所以sql可以是非常简单的：

```
SELECT * FROM `tree` WHERE `left` > 1 AND `right` < 10;

```

对于一个节点到底有多少个子孙， 其实这个问题很简单，只需要将节点(right - left - 1) / 2即可。


#### 获取节点深度


对c# programmer这个节点，对比于boss、c# leader 和java leader就可以发现，
只要左边比c# programmer 小，但右边比c# programmer大的节点都是到达c# programmer路径上的，所以只需要统计这些节点的数量就可以知道c# programmer的深度。

```

DELIMITER $$

CREATE FUNCTION NodeLayer(node_id INT)
RETURNS INT

BEGIN
    DECLARE layer INT;
    DECLARE left INT;
    DECLARE right INT;
    SET layer = 0;
    -- get node's left, right
    SELECT `left`, `right` INTO left, right FROM `tree` WHERE `id` = node_id;
    -- count layer
    SELECT COUNT(*) INTO layer FROM `tree` WHERE `left` <= left AND `right` >= right;
    RETURN (layer);
END $$

DELIMITER ;

```

#### 获取某节点的族谱路径

根据 ‘获取节点深度’ 分析可知，需要获取某节点的族谱路径只需要分析节点左右值，然后一条sql就可以解决了：

```
DELIMITER $$

CREATE FUNCTION NodeLayer(node_id INT)
RETURNS INT

BEGIN
    DECLARE left INT;
    DECLARE right INT;
    -- get node's left, right
    SELECT `left`, `right` INTO left, right FROM `tree` WHERE `id` = node_id;
    -- count layer
    SELECT * FROM `tree` WHERE `left` < left AND `right` > right ORDER BY `left` ASC;
END $$

DELIMITER ;

```

#### 加入节点

加入一个节点也是很简单的， 但如果树的规模比较大的时候需要修改的节点比较多，所以开销也是比较大的。先上图：

![add_tree_node.png][2]




































[1]: http://www.spoofer.top/assets/images/2017/07/树形.png
[2]: http://www.spoofer.top/assets/images/2017/07/add_tree_node.png
