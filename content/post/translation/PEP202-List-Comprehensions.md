---
title: "[译] PEP202 - List Comprehensions 列表推导式"
description: 本篇 PEP 描述了一项 Python 语法扩展建议，列表推导式
date: 2023-01-09T17:00:01+08:00
image: https://raw.githubusercontent.com/Arrackisarookie/images/main/pep.png
math: 
license: 
hidden: false
comments: true
categories:
    - 译文
tags:
    - Python
---

> 原文标题：PEP 202 List Comprehensions  
> 原文作者：Barry Warsaw \<barry at python.org\>  
> 原文链接：[https://peps.python.org/pep-0202/](https://peps.python.org/pep-0202/)

## 概览
本篇 PEP 描述了一项 Python 语法扩展建议，列表推导式。

## 方案
建议列表内元素可以使用 `for` 和 `if` 语句进行条件构造（包括相应嵌套格式）。

## 基本原理
现今，使用 `map()`、`filter()` 配合嵌套循环创建列表的方式大行其道，而列表推导式提供了一种更简便的方式。

## 例子
``` python
>>> print [i for i in range(10)]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

>>> print [i for i in range(20) if i%2 == 0]
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

>>> nums = [1, 2, 3, 4]
>>> fruit = ["Apples", "Peaches", "Pears", "Bananas"]
>>> print [(i, f) for i in nums for f in fruit]
[(1, 'Apples'), (1, 'Peaches'), (1, 'Pears'), (1, 'Bananas'),
 (2, 'Apples'), (2, 'Peaches'), (2, 'Pears'), (2, 'Bananas'),
 (3, 'Apples'), (3, 'Peaches'), (3, 'Pears'), (3, 'Bananas'),
 (4, 'Apples'), (4, 'Peaches'), (4, 'Pears'), (4, 'Bananas')]
>>> print [(i, f) for i in nums for f in fruit if f[0] == "P"]
[(1, 'Peaches'), (1, 'Pears'),
 (2, 'Peaches'), (2, 'Pears'),
 (3, 'Peaches'), (3, 'Pears'),
 (4, 'Peaches'), (4, 'Pears')]
>>> print [(i, f) for i in nums for f in fruit if f[0] == "P" if i%2 == 1]
[(1, 'Peaches'), (1, 'Pears'), (3, 'Peaches'), (3, 'Pears')]
>>> print [i for i in zip(nums, fruit) if i[0]%2==0]
[(2, 'Peaches'), (4, 'Bananas')]
```

## 参考实现
列表推导式已成为 Python 语言 2.0 版本的一部分，相关信息已收录至[1](http://docs.python.org/reference/expressions.html#list-displays)

## BDFL 声明
+ 请参照以上提出的语法
+ 不支持 `[x, y for ...]` 格式，应调整为 `[(x, y) for ...]` 格式
+ 嵌套格式 `[... for x ... for y ...]` 中，类似普通 for 嵌套循环，最后面的索引最先执行

## 参考
[1 http://docs.python.org/reference/expressions.html#list-displays](http://docs.python.org/reference/expressions.html#list-displays)

源文件：[https://github.com/python/peps/blob/main/pep-0202.txt](https://github.com/python/peps/blob/main/pep-0202.txt)