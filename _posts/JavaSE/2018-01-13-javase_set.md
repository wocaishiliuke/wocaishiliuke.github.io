---
title: Set
date: 2018-01-13 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Collection的子接口Set的相关成员。

<!-- more -->

##### 目录
- [X] I.Set
- [X] II.HashSet


---

# I.Set

Set是Collection的子接口。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/Set.png)

- 无索引
- 元素唯一（List的add()永远返回true，而Set的可能是false）
- 无序，存取不一致
- 允许为null值（只能有一个）

> Set中没有特殊的方法，和Collection一样。Java中的Set跟数学中的集合一致（无序性、互斥性、确定性）。

**Set底层依赖Map，Set保证元素唯一的方式，也是Map保证key唯一的方式**。

> Set接口也有抽象层AbstractSet类，实现一些通用方法。


---

# II.HashSet