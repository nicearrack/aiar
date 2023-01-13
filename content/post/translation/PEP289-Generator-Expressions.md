---
title: "[译] PEP289 - Generator Expressions 生成式表达式"
description: 本篇 PEP 介绍了生成器表达式(Generator Expressions)这种高性能的，内存高效泛化的列表推导式(List Comprehensions)和生成器(generators)。
date: 2021-05-21T09:03:49+08:00
image: https://raw.githubusercontent.com/Arrackisarookie/images/main/tech/pep.png
math: 
license: 
hidden: false
comments: true
categories:
    - 译文
tags:
    - Python
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

## 细节描述
（也许对于火星读者来说，下面这些说的可能没那么全面确切，但我相信这些例子已经足够表述我的想法，以便在 c.l.py 进行讨论(译者注：最后半句没看懂)。Python 参考手册应涵盖以下所有语义语法规格。）

1. 生成器表达式的语义等同于创建一个匿名生成器函数并调用它。例如：
``` python
>>> g = (x**2 for x in range(10))
>>> print g.next()
```
这等同于：
``` python
>>> def __gen(bound_exp):
...   for var1 in bound_exp:
...     if exp2:
...       for var2 in exp3:
...         if exp4:
...           yield tgtexp

>>> g = __gen(iter(exp1))
>>> del __gen
```

2. 句法要求，生成器表达式需要直接置于一组小括号内，并且两边都不能有逗号。参考 CVS 中的 `Grammar/Grammar` 文件，两条规则更改(*译者注：语法文件可参照[官方文档](https://docs.python.org/3.11/reference/grammar.html)*)：

+ 规则1
```
atom: '(' [testlist] ')'
```
更改为：
```
atom: '(' [testlist_gexp] ')'
```
其中 testlist_gexp 和 listmaker 基本没差，但 testlist_gexp 只允许在 `for ... in` 之间进行单个测试。
```
testlist_gexp: test ( gen_for | (',' test)* [','] )
```

+ 规则2，参数列表的规则需要进行类似修改。

也就意味着，只有一个参数时，生成器表达式小括号可以省略，如：
``` python
>>> sum(x**2 for x in range(10))
```

但其他情况下，表达式小括号必须填写：
``` python
>>> reduce(operator.add, (x**2 for x in range(10)))
>>> g = (x**2 for x in range(10))
```

确切的细节已签入 `Grammar/Grammar` 1.49版。

3. 如果将一个或一组简单变量作为循环变量，那么它(们)不会暴露给外层函数。这样一来不仅有助于开发编码，而且可以让典型用例更加可信赖。在Python的一些未来版本中，列表推导式的迭代变量也将对外层函数代码隐身（并且在Py2.4中，访问迭代变量将会触发Warning）。例如：
``` python
>>> x = 'hello'
>>> y = list(x for x in 'abc')
>>> print x
hello  # 而不是 c
```

4. 列表推导式语法将保持不变，例如：
``` python
>>> [x for x in S]  # 这是一个列表推导式
>>> [(x for x in S)]  # 这是一个列表，里面包含了一个生成器表达式
```
不幸的是，目前二者有着轻微的句法差异，列表推导式：
``` python
>>> [x for x in 1, 2, 3]  # python3 已不支持该写法
```
是合法的，它等同于：
``` python
>>> [x for x in (1, 2, 3)]
```
但是生成器表达式不支持以上形式：
``` python
>>> (x for x in 1, 2, 3)
```
非法。

之前的列表推导式语法将在 Python 3.0 中不再支持，同时也会在 Python2.4 及以后版本中被标为弃用。

为了让 Python3.0 中列表推导式的语义定义等同于 `list(<generator expression>)`，上面提到的列表推导式会将其迭代变量“泄漏”到外层环境中的问题，也将在 Python 3.0 中得到修复。同时，在Py2.4 及以后版本中，如果列表推导式的迭代变量和当前上下文中的变量重名时，解释器将触发弃用警告。

## 早期绑定 vs. 后期绑定
待续

## 名词解释
[1] BDFL 终身仁慈独裁者（英语：Benevolent Dictator For Life，缩写BDFL）是少数开源软件开发者所拥有的头衔。他们通常是某一项目的创始人，并在该项目社区出现争议时拥有最终的决定权。来自[维基百科](https://en.wikipedia.org/wiki/Benevolent_dictator_for_life)
