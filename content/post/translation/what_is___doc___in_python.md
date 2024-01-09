---
title: "[译] Python中的__doc__是啥"
description: Python 的每个对象中都有一个叫做 `__doc__` 的属性，用来存放该对象的文档信息。比如对于 `Dog` 类，可以直接调用 `Dog.__doc__` 来获取它的文档字符串(docstring)信息
date: 2023-01-30T09:56:47+08:00
image: https://s11.ax1x.com/2024/01/08/pFShAdH.png
comments: true
categories:
    - 译文
tags:
    - Python
---

> 原文标题：What is __doc__ in Python?  
> 原文作者：[Chris](https://blog.finxter.com/author/xcentpy_cfsh849y/)  
> 原文链接：[https://blog.finxter.com/what-is-__-doc-__-in-python/](https://blog.finxter.com/what-is-__-doc-__-in-python/)

## What

Python 的每个对象中都有一个叫做 `__doc__` 的属性，用来存放该对象的文档信息。比如对于 `Dog` 类，可以直接调用 `Dog.__doc__` 来获取它的文档字符串(docstring)信息。

可以使用三个引号将字符串包围的方式来定义文档字符串，就像例子中这样：

``` python
class Dog:
    """你最好的朋友。"""

    def do_nothing(self):
        pass


print(Dog.__doc__)
# 你最好的朋友。
```

Python 中万物皆对象，函数也不例外，所以也可以在函数中定义文档字符串：

``` python
def bark():
    """汪汪汪"""
    pass


print(bark.__doc__)
# 汪汪汪
```

注意，如果没有定义文档字符串，那么调用 `xxx.__doc__` 时将返回 `None`。

``` python
def bark():
    pass


print(bark.__doc__)
# None
```

## Why
为什么要使用文档字符串(docstring)呢？

在代码中定义文档字符串最大的好处在于，可以以编程的方式创建漂亮的文档了。借助类似 [Sphinx](https://www.sphinx-doc.org/en/master/examples.html) 这样的工具，为项目创建类似下图这样的文档将变得非常容易，只需要在代码中定义文档字符串，即为 `__doc__` 赋值。

![Sphinx-doc](https://s11.ax1x.com/2024/01/08/pFShMy8.png)

## Practice
[官方 PEP 标准](https://www.python.org/dev/peps/pep-0257/)中定义了很多文档字符串优雅的实践，它们被称为 *文档字符串规范(Docstring Conventions)*。定义项目中的文档字符串时，请尽可能按照这些规范。下面将列举出规范中最重要的7条：

1. 所有的模块(module)，函数(function)，方法(method)和类(class)都应该拥有文档字符串
2. 为了一致性原则，请使用 `"""三个双引号"""` 来包围文档字符串
3. 即使文档字符串一行就能写下也应使用三个引号，以便于以后扩展
4. 若无特殊情况，文档字符串前后请不要有空行
5. 描述项目代码的行为时，请使用类似 `"""Do X and return Y."""` 的主格形式，然后以句点结尾。请 **不要使用** 类似 `"""Does X and returns Y."""` 这样的第三人称单数形式
6. 多行文档字符串可以以一句概括开头，然后一个空行，接着是更详细的描述，类似 `argument --- name of the person (string)` 这样来描述函数或方法的一个参数，每个参数占一行。
7. 多行文档字符串无需另起一行，紧接着引号开始就好，就像这样 `"""Some summary...`

如果你是个完美主义者，或是有了中级的代码能力，可以查看[官方文档](https://www.python.org/dev/peps/pep-0257/)来获取更多的例子。
