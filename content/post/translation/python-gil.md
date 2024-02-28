---
title: "[译] Python全局解释器锁(GIL)是什么"
slug: 37e820d83e914ea38386ad1aa323f3c5
description: 在这篇文章中，你将会学习到GIL是如何影响Python程序性能的，以及如何缓解这些影响
date: 2024-02-27T10:16:37+08:00
image: https://s11.ax1x.com/2024/02/28/pFdbDPK.jpg
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - 译文
tags:
  - GIL
  - Python
---

> 原文标题：What Is the Python Global Interpreter Lock (GIL)?  
> 原文作者： Abhinav Ajitsaria  
> 原文链接：[https://realpython.com/python-gil/](https://realpython.com/python-gil/)

简单来说，Python 的全局解释器锁(GIL)是一种互斥元(或锁)，同一时刻它只允许一个线程持有对 Python 解释器的控制权。

这意味着，在任意一个时间点都只能有一个线程处于执行状态。GIL 带来的影响对于只执行单线程程序的开发者几乎不可见，但是它会成为计算密集型或多线程代码的性能瓶颈。

由于即使在拥有多个 CPU 的多线程架构中，GIL 也只允许同一时刻仅有一个线程可执行，因此 GIL 已经臭名昭著了。

**在这篇文章中，你将会学习到GIL是如何影响Python程序性能的，以及如何缓解这些影响**

## GIL 解决了 Python 中 的什么问题
Python 使用引用计数进行内存管理，这意味着每一个在 Python 中创建的对象都有一个引用数变量，用于保持追踪有多少指向该变量的引用。当这个数量变为零时，该变量占用的内存将被释放。（译者注：详见[CPython 的垃圾回收](https://aiar.site/post/52397c9cfe524c17a37e15b83a022f94/)）

来看一段代码演示下引用计数是如何工作的：
```python
>>> import sys
>>> a = []
>>> b = a
>>> sys.getrefcount(a)
3
```
在上面的例子中，空列表对象 `[]` 的引用计数是 3。该对象被 `a`、`b` 以及传入 `sys.getrefcount()` 方法的参数所引用。

回到 GIL：

问题在于，这个引用计数变量需要避免竞争局面的产生，比如两个线程同时增加或者减少这个值。这种局面一旦出现，轻则造成一部分内存永远不会被释放，重则导致一些还拥有引用的对象的内存被释放。而这可能会导致 Python 程序崩溃，或者其他奇奇怪怪的 bug。

比较容易想到的解决方案是，为所有跨线程共享的数据结构加锁，这样可以通过保证数据结构修改的一致性，进而保证引用计数变量的安全性。

但是，为每一个对象或每一组对象加锁意味着会有多个锁同时存在，这将造成另外一个问题——死锁(死锁只会发生在同时存在超过一个锁的情况下)。从另一个角度来看，频繁申请和释放锁资源也会降低程序性能。

GIL 是添加在解释器自身上的一个简单的锁，它有这样一个规则：执行任何字节码都需要申请解释器锁。这防止了死锁(因为只有一个锁存在)，也不会对性能造成过多的影响。但是它实际上造成任何计算密集型 Python 程序只能是单线程。

尽管 GIL 也被其他语言的解释器使用，比如 Ruby，但它并不是这个问题唯一的解决方案。一些语言通过使用引用计数之外的方法(如垃圾回收)来避免 GIL 对线程安全内存管理的依赖。

另一方面，这也意味着这些语言必须通过添加其他性能提升功能（如JIT编辑器）来弥补 GIL 单线程性能优势的损失。

## 为什么 GIL 能被选为解决方案
那么，到底为什么这样一个看起来如此不堪的解决方案会被用在 Python 里呢？这是 Python 开发人员的一记昏招么？用 [Larry Hastings 的话](https://youtu.be/KVKufdTphKs?t=12m11s)(译者注：Python的一位核心开发人员和长期使用者)来说，GIL 的设计决策是 Python 在今天如此受欢迎的原因之一。

在操作系统还没有线程这个概念时，Python 就已经诞生了。为了让开发更加的快捷，Python 以易用作为开发理念，这也使得越来越多的开发者开始使用 Python。

很多扩展都是针对已有的 C 语言库来编写的，这些库都是 Python 所必需的。为了确保更改的一致性，这些 C 扩展需要一个线程安全的内存管理机制，而这正是 GIL 可以提供的。

GIL 很容易实现，也很容易添加到 Python 中。由于只需要管理一个锁，它也提升了单线程的程序的性能。

非线程安全的 C 库变得更容易集成，而 Python 也因为这些 C 扩展，变得更容易被不同社区采用。

如你所见，GIL 是 CPython 的开发人员在 Python 早期面临这个难题时的实用性解决方案。

## 对多线程的 Python 程序的影响
对于一个典型的 Python 程序甚至任何一个相关的计算机程序来说，计算密集型和 I/O 密集型在性能要求上是有区别的。

计算密集型程序倾向于将 CPU 的运算能力运用到极致，像多维矩阵数学计算、搜索、图像处理等等都属于计算密集型程序。

I/O 密集型程序是那些需要花费时间等待输入输出的程序，这些输入输出可能来自于用户、文件、数据库、网络等等。I/O 密集型程序有时不得不消耗大量时间用于等待，直到它们从源端取到所需。这是由于在源端输入输出准备好之前，它可能自己也需要做一些处理，比如，用户在思考到底要输入什么指令，或者数据库在查询时运行自己的进程。

来看一个简单的计算密集型程序，它实现了一个记录倒数时长的功能：
```python
# single_threaded.py
import time

COUNT = 50000000

def countdown(n):
    while n>0:
        n -= 1

start = time.time()
countdown(COUNT)
end = time.time()

print('Time taken in seconds -', end - start)
```

在我这台 4 核的机器上执行它会有这样的输出：

```shell
$ python single_threaded.py
Time taken in seconds - 6.20024037361145
```

现在，简单调整下代码，使用两个线程分开执行一半的倒数任务：
```python
# multi_threaded.py
import time
from threading import Thread

COUNT = 50000000

def countdown(n):
    while n>0:
        n -= 1

t1 = Thread(target=countdown, args=(COUNT//2,))
t2 = Thread(target=countdown, args=(COUNT//2,))

start = time.time()
t1.start()
t2.start()
t1.join()
t2.join()
end = time.time()

print('Time taken in seconds -', end - start)
```

然后再次执行：
```shell
$ python multi_threaded.py
Time taken in seconds - 6.924342632293701
```

如你所见，两个版本的程序基本花费了相同的时间(甚至多线程的还更长些)。在多线程版本中，GIL 阻止了计算密集线程的并行执行。

GIL 对于 I/O 密集型的多线程程序在性能方面影响甚微，因为锁在等待 I/O 时是共享的。

但是那些纯粹的计算密集型程序，比如一个使用多线程将图片分为几部分的程序，不仅会在锁的影响下变成单线程，而且就像上面的例子那样，你还会看到相对于单线程程序，多线程的执行时间还增加了。

这种时间的增加，就是线程锁在获取和释放时的性能开销所致。

## 为什么 GIL 还没有被移除
Python 的开发团队收到过数不胜数的关于 GIL 的抱怨，但是像 Python 这样流行的语言，如果做出像移除 GIL 这样重大的改变，势必会造成一系列向下兼容的问题，而这是 Python 团队无法接受的。

退一万步说，GIL 完全可以被移除，之前开发人员和研究人员也曾尝试过很多次，但是所有的尝试都破坏了现有的 C 扩展，因为这些扩展都严重依赖 GIL 提供的解决方案。

当然，是有一些其他的 GIL 替代方案，但是它们有些会降低单线程应用和 I/O 密集型多线程程序
的性能，同时这些程序也会变得异常复杂。毕竟，你也不想在 Python 新版本出来后导致已有的程序效率变慢吧？

Python 的创始人、BDFL，Guido van Rossum，于 2007 年九月在社区发布的[《移除GIL并不容易》](https://www.artima.com/weblogs/viewpost.jsp?thread=214235)一文中做出了一些回应：
> “I’d welcome a set of patches into Py3k only if the performance for a single-threaded program (and for a multi-threaded but I/O-bound program) does not decrease”
> “只要单线程程序(以及多线程的 I/O 密集型程序)的效率不会下降，我是非常欢迎为 Py3k 附加一些补丁的”

但自那以后，任何一次尝试都没有满足这个条件。

## 为什么 Python 3 也没有移除它
Python 3 确实有机会从头开始开发许多特性，但在这个过程中也破坏了一些现有的 C 扩展，这些扩展需要更新和移植才能与 Python 3 一起工作。这也是在 Python 3 的早期版本社区采用较慢的主要原因。

但是为啥 GIL 还是没有被移除呢？

移除 GIL 就会导致新的 Python 3 版本在单线程程序的性能方面相比于 Python 2 竟然会更慢，可想而知结果会是啥。我们确实无法否认 GIL 在单线程程序性能上的优势，因此结果就是 Python 3 中的 GIL 仍然存在。

但是，Python 3 也确实对现有的 GIL 进行了重大的改进——

我们之前讨论了 GIL 对于纯计算密集型或纯 I/O 密集型多线程程序的影响，但是，对于那些一部分线程计算密集，另一部分线程 I/O 密集的程序 GIL 带来的影响又会是怎样呢？

在这样的程序中，Python 的 GIL 会饿死 I/O 密集型的线程，不给它们从计算密集型线程这种获取 GIL 的机会。

首先我们需要知道 Python 有这样一个内置的机制，当某线程连续使用 GIL 到达一个 **固定的时长** 后，机制会强制让线程释放 GIL，但如果没有别的线程申请 GIL，则该线程就会继续使用 GIL。
```python
>>> import sys
>>> # The interval is set to 100 instructions:
>>> sys.getcheckinterval()
100
```

但这个机制的问题在于，对于大部分的计算密集型线程而言，它们会抢在别的线程能申请之前重复申请 GIL。这项研究由 David Beazley 整理，可以在 [这里](https://www.dabeaz.com/blog/2010/01/python-gil-visualized.html) 找到可视化的成果。

这个问题于 2009 年由 Antoine Pitrou 在 Python 3.2 中修复，他添加了一种机制用来监测其他线程请求获取 GIL 但被丢弃的次数，并且在其他线程有机会运行之前，不再允许当前线程重复获取 GIL。

## 如何来处理 Python 的 GIL
如果 GIL 正在给你添堵，可以尝试用这些方法解决：

**多进程 vs 多线程：** 最受欢迎的方式是使用多进程代替多线程。每个 Python 进程都有它自己的解释器和内存空间，所以 GIL 也就不是问题了。Python内置的 [multiprocessing](https://docs.python.org/3/library/multiprocessing.html) 模块(译者注：原始链接是Python2的，这里的链接换成了最新的Python版本的)可以帮我们很容易的创建进程，就像这样：
```python
from multiprocessing import Pool
import time

COUNT = 50000000
def countdown(n):
    while n>0:
        n -= 1

if __name__ == '__main__':
    pool = Pool(processes=2)
    start = time.time()
    r1 = pool.apply_async(countdown, [COUNT//2])
    r2 = pool.apply_async(countdown, [COUNT//2])
    pool.close()
    pool.join()
    end = time.time()
    print('Time taken in seconds -', end - start)
```
在我的机器上执行会有这样的输出：
```shell
$ python multiprocess.py
Time taken in seconds - 4.060242414474487
```
相较于多线程版本，这样能有一个不错的性能提升，还可以吧~

我们看到，时间花费并没有下降为原本的一半，这是因为进程管理也有它自己的花费。多进程比多线程更重，所以要考虑清楚，这可能会成为新的瓶颈。

**使用其他解释器：** Python 有很多解释器的实现版本，最受欢迎的有 CPython、Jython、IronPython 以及 PyPy，它们分别是由 C、Java、C# 和 Python 实现的版本。GIL 仅存在于最初的实现版本 CPython 中。如果你的程序以及它依赖的一些库对于一种或多种解释器都适用，那么你也可以尝试使用别的解释器。

**敬候佳音：** 尽管有很多 Python 的使用者在利用 GIL 在单线程程序上的性能优势，但是多线程开发人员也不用太过发愁，因为 Python 社区中一些最聪明的人正致力于将 GIL 从 CPython 中移除，比如 [Gilectomy](https://github.com/larryhastings/gilectomy) 这个尝试。(译者注：可惜这个尝试已经有8年没有更新了，最后截至到Python3.6版本)

Python 的 GIL 经常被看作是一个神秘而又困难的主题，但作为 Pythonista，通常你只会在写 C 扩展或者计算密集型程序时，才会被它影响。

因此，本文为你介绍了 GIL 是什么以及在程序中如何处理它。如果你想继续了解 GIL 底层工作原理，建议观看 David Beazley 的[《理解 Python GIL》](https://youtu.be/Obt-vMVdM8s)这场演讲。
