---
title: "[译] PEP289 - Generator Expressions 生成式表达式"
description: 本篇 PEP 介绍了生成器表达式(Generator Expressions)这种高性能的，内存高效泛化的列表推导式(List Comprehensions)和生成器(generators)。
date: 2021-05-21T09:03:49+08:00
image: 
math: 
license: 
hidden: false
comments: true
categories:
    - Tech
tags:
    - Python
    - Translation
---

> 原文标题：PEP 289 - Generator Expressions  
> 原文作者：python at rcn.com (Raymond Hettinger)  
> 原文链接：[https://peps.python.org/pep-0289/](https://peps.python.org/pep-0289/)

## 概览
本篇 PEP 介绍了生成器表达式(Generator Expressions)这种高性能的，内存高效泛化的列表推导式(List Comprehensions)和生成器(generators)。

## 基本原理
从以往编程经验来看，列表推导式在 Python 的各个角落都有广泛的实用性。但是，其中不少的情况其实并不需要在内存中创建完整的列表(List)，而一次迭代一个元素就恰好满足它们的需求。

例如，下面这段列表求和的代码将在内存中完整的创建一个乘方的列表，然后遍历其中每一个值，最后当该引用(reference)不再需要时，删除整个列表：
``` python
sum([x * x for x in range(10)])
```

现在，可以通过使用生成器表达式来节省内存：
``` python
sum(x * x for x in range(10))
```

其他容器对象的构造函数也同样支持类似的特性：
``` python
s = set(word for line in page for word in line.split())
d = dict((k, func(k)) for k in keylist)
```

生成器表达式对于类似 `sum()`，`min()`， `max()` 这种能将一个可迭代的输入汇聚成一个值的函数有着非常高效的作用：
``` python
max(len(line) for line in file if line.strip())
```

生成器表达式简化(address)了一些使用 `lambda` 函数的例子：
``` python
reduce(lambda s,.a: s + a.myattr, data, 0)
reduce(lambda s,.a: s + a[3], data, 0)
```
它们可以被简化为：
``` python
sum(a.myattr for a in data)
sum(a[3] for a in data)
```

列表生成式极大减少了开发人员对 `filter()` 和 `map()` 的需求；同时，生成器表达式也被寄予厚望来最大可能的减少人们对 `itertools.ifliter()` 和 `itertools.imap()` 的使用需求。相比之下，`itertools` 中其他方法的能力将被生成器表达式进一步增强：
``` python
dotproduct = sum(x * y for x, y in itertools.izip(x_vector, y_vector))
```

在升级扩展应用时，与列表推导式类似的语法也能更容易的将已有代码转换为生成器表达式。

早期的版本中，生成器表达式相比于列表推导式有着相当明显的性能优势。但是后者针对 `Py2.4` 做了高度优化，现在二者在处理中小型数据集时的性能已经大致相当。

但随着数据量的增长，由于生成器表达式不会耗尽缓存内存，同时还允许 Python 在迭代之间复用对象，所以生成器表达式往往能展现出更好的性能。

## BDFL 声明
本 PEP 已被 Py2.4 接受。[[1]](#名词解释)



## 名词解释
[1] BDFL 终身仁慈独裁者（英语：Benevolent Dictator For Life，缩写BDFL）是少数开源软件开发者所拥有的头衔。他们通常是某一项目的创始人，并在该项目社区出现争议时拥有最终的决定权。来自[维基百科](https://en.wikipedia.org/wiki/Benevolent_dictator_for_life)
