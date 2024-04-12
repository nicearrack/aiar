---
title: "[译] 使用并发加速你的程序"
slug: 423359792f6842499cae60e1b2595c5a
description: 
date: 2024-02-29T17:39:06+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
---

> 原文标题：Speed Up Your Python Program With Concurrency  
> 原文作者：Jim Anderson  
> 原文链接：[https://realpython.com/python-concurrency/](https://realpython.com/python-concurrency/)

如果你曾听过很多关于 `asyncio` 即将加入 Python 标准库的讨论(译者注: asyncio于Python3.4时被引入标准库，于Python3.6基本稳定)，但很是好奇它和其他并发方法相比有何异同；或者想知道并发到底是什么，以及它是如何为你的程序加速的；那你就来对地方啦~

**在本文中，你会了解到以下知识：**
+ 什么是 **并发**
+ 什么是 **并行**
+ Python 中一些 **并发方法** 的比较，包括线程、进程、异步I/O
+ **何时** 需要在程序中 **使用并发**，用哪个模块

本文需要读者对 Python 有一些基本的理解。文中代码示例最低可以使用 Python 3.6 来运行。读者可以从 [RealPython Github repo](https://github.com/realpython/materials/tree/master/concurrency-overview) 中下载这些代码。

## 什么是并发
字典中对于并行的定义是，同时发生。在 Python 中，对于同时发生的事情有很多个称谓(线程、任务、进程)，但是从本质上看，它们都指代的是一系列按顺序运行的指令。

我喜欢把它们比作是不同的“想法”，每一个想法都可以在某个时间点停下，然后处理它们的 CPU 或大脑改为“思考”另一个想法。每个想法的状态都会被保存下来，以便于它们再被“想起”时可以从上次中断的地方开始。

你可能想知道为什么 Python 对于同一个概念会有不同的称谓，但事实上，线程、任务、进程只有从本质上看时才是相同的。一旦你开始深挖细节，会发现它们其实代表的并不是同一种事物。跟随本文的脚步，你会发现它们之间更多的不同。

现在，我们开始讨论定义中“同时”的含义。当我们深入了解其细节时必须要严谨一些，因为只有 `多进程` 才能真正的同时“思考”多个“想法”。线程(threading 库)和异步(asyncio 库)底层还是在同一个处理器上运行，因此同一时刻还是只能运行一个。它们只是运用一些巧妙的方式提升了依次执行的效率，进而加速整个处理过程。所以尽管它们并没有真正的让机器同时“思考”不同的“想法”，但我们也仍会称它们为并发。

多线程和异步最大的区别在于它们处理线程和任务顺序执行时使用的方式。对于多线程来说，操作系统清楚地了解每一个线程，也可以随时中断一个线程并启动另一个。正是由于操作系统具有这种可以随时剥夺一个线程执行权然后直接给另一个的能力，所以这种方式被称为抢占式多任务处理。

抢占式多任务处理非常方便，因为线程中的代码不需要为CPU控制权转换做任何事情。但是，代码也可能会因为“随时”而变得非常复杂。因为线程切换可能发生在任意一个独立的 Python 语句中间，即使是 `x = x + 1` 这种微不足道的语句。

而对于异步I/O来讲，它使用的是协作式多任务处理。多个任务之间若要实现协作，就必须宣告它们自己何时退出占用。这也意味着，相比于顺序执行，多任务中的代码必须稍微调整一下才能实现这一点。

提前做这些额外工作的好处就在于，开发人员总能知道任务会在哪里退出占用。只要一段 Python 程序没有被标记，那么它就绝不会在中间退出占用。在后面的章节中，我们会继续介绍异步是如何简化部分设计的。

## 什么是并行
看到这里，我们已经介绍了在单核处理器上发生的并发。那么，你那酷酷的新笔记本电脑所拥有的多核处理器会有怎样的船新体验呢？你又该如何使用它们呢？多进程告诉你答案。

Python 可以通过 `multiprocessing` 库创建新的进程。虽然从技术角度上看，进程被定义为一系列像内存、文件句柄这样的资源集合，而这里的进程可以被认为是一个完全不同的程序。也就是说我们可以这么想，每一个进程在它自己的 Python 解释器中运行。

因为是它们是不同的进程，所以在多进程程序中的每一个“思路”都可以在不同的处理器核中运行。可以在不同核中运行也就意味着可以真正做到让它们在相同时刻运行，如同天上降魔主，真是人间太岁神，这个多进程__(发送评论查询译者精神状况ヾ(≧▽≦*)o)。虽然这样做会增加一些复杂性，但是 Python 在大多数情况下都能很好的解决这些问题。

想必你对并发和并行已经有了些许概念，那现在我们来复习一下它们的区别，后面我们再看看它们究竟为什么这么有用：

| 并发类型                 | 执行权的决定因素      | 处理器数量 | 
|----------------------|---------------|-------|
| 抢占式多任务处理(threading)  | 由操作系统决定何时切换   | 1     |
| 协作式多任务处理(asyncio)    | 由任务本身决定和放弃控制权 | 1     |
| 多进程(multiprocessing) | 进程们同时在多个核上运行  | 很多    |

这些并发都很有用，接下来我们来看看它们各自会在什么情况下能提升速度。

## 并发什么时候有用
有两类问题在面对并发时会有非常大的不同，通常它们被称为计算密集型和 I/O 密集型问题。

I/O 密集型问题造成程序速度慢的原因在于它必须频繁的等待外部资源的输入或输出。当你的程序和一些比 CPU 慢很多的事物一起运作时，这种问题就会频繁出现。

比 CPU 慢的事物数不胜数，但好在它们大部分都不会和你的程序有交互。而在剩下的那些当中，和程序交互最频繁的就是文件系统和网络连接了。

下图将这种交互描绘的非常形象：
![I/O密集型交互](https://robocrop.realpython.net/?url=https%3A//files.realpython.com/media/IOBound.4810a888b457.png&w=868&sig=f1a9d0fbb4e7f743a55ee0fec76eb627cfbf3571)

图中蓝色的方块表示程序执行消耗的时间，红色的方块表示等待 I/O 操作直到完成所消耗的时间。方框大小的比例并不指代真实的耗时比例，因为网络请求的耗时会比 CPU 指令多几个数量级，因此你的程序可能大部分时间都花在等待上了。这也是浏览器这种程序大部分时间在做的事情hhh

在另一个极端，有些程序并没有多少网络请求或者文件访问，但需要做大量的运算。由于这些程序限制速度的瓶颈在于 CPU 的运算效率而不是网络连接或文件系统，所以这类程序就被称为计算密集型程序。

下图是计算密集型程序相应的图：
![计算密集型程序](https://robocrop.realpython.net/?url=https%3A//files.realpython.com/media/CPUBound.d2d32cb2626c.png&w=868&sig=88b3c863f684b8109ed17e4a013aab1e666dba1f)

在下一节的示例中，你将看到不同种类的并发在计算密集型和I/O密集型程序中优劣。在程序中使用并发会增加额外代码和复杂度，所以你需要权衡潜在的速度提升是否值得花费这些额外的代价。等你看到文章的结尾，也许就有足够的信息来做权衡了。

这里是一些解释概念的的简要概述：

| I/O 密集型程序                              | 计算密集型程序               |
|----------------------------------------|-----------------------|
| 程序的大部分时间花费在和相对较慢的设备做交互上，比如网络连接，硬盘或者打印机 | 程序的大部分时间花费在做 CPU 运算上  |
 | 提升速度需要尽可能将不同设备的等待时间重叠在一起               | 提升速度需要让程序在相同时间内做更多的运算 |

接下来我们先介绍 I/O 密集型程序再看计算密集型程序。

## 如何提升I/O密集型程序的速度
我们来聚焦一下 I/O 密集型程序面临的一个常见的需求：从网络上下载。我们的例子实现了从一些站点中下载 web 页面的功能，当然，任何网络资源都可以，下载页面只是因为它更容易展示和启动。

### 同步版本
对于这个任务，我们先从非并发版本开始。需要注意的是，这个程序依赖 [requests](http://docs.python-requests.org/en/master/) 库，即运行之前需要执行 `pip install requests`。这个版本一点并发都没用：
```python
import requests
import time


def download_site(url, session):
    with session.get(url) as response:
        print(f"Read {len(response.content)} from {url}")


def download_all_sites(sites):
    with requests.Session() as session:
        for url in sites:
            download_site(url, session)


if __name__ == "__main__":
    sites = [
        "https://www.jython.org",
        "http://olympus.realpython.org/dice",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} in {duration} seconds")
```

如你所见，这程序真的相当短小了。`download_site()` 函数仅仅是下载了 URL 里面的内容，同时打印了内容的大小。值得注意的一点是，我们使用了 `requests` 库中的 [Session](https://2.python-requests.org/en/master/user/advanced/#id1) 对象。

直接简单的使用 `requests` 中的 `get()` 函数当然也可以，但是创建一个 `Session` 对象能让 `requests` 做一些有趣的网络技巧，还能提升速度。

`download_all_sites()` 函数先创建了 `Session` 对象，然后遍历 sites 列表再依次下载。最后它打印出整个过程的耗时，这样在后面的例子中，我们就可以欣慰的看到使用并发能带来多大的帮助。

这个程序的流程图和上一小节的 I/O 密集型程序的图非常相似。

> 注意：影响网速的因素有很多，甚至每秒都在变化。测试的时候我曾遇见过因为网速的原因导致运行时间多了一倍。

#### 为什么同步版本如此流行
这个版本的伟大之处就在于，它非常简单。开发人员可以更容易的编写和调试，考虑问题时也能更直接。只有一条思路贯穿始终，我们就可以预测下一步是什么以及它会有怎样的表现。

#### 同步版本的缺陷
最大的问题在于，相对于后面提供的其他解决方案，同步版本慢得可怕。下面是在我的机器上运行的输出；
```shell
$ ./io_non_concurrent.py
   [most output skipped]
Downloaded 160 in 14.289619207382202 seconds
```
> 注意：你的结果可能和这个出入很大，运行脚本时耗时大概在 14.2 - 21.9 秒左右。本文中我使用了三次实验中最快的一个数值，但和其他方式相比，差距还是十分明显。

其实程序运行的相对较慢不是什么大事，如果你的程序使用同步版本只用了2秒，而且运行的次数也非常稀少，那可能加并发也没太大必要。也就没必要再往下看了

那如果程序运行的非常频繁呢？如果运行一次就要好几个小时呢？来来来，移步到并发这边，我们使用 `threading` 重新一下刚刚的程序。

### threading 多线程版本
你可能会想，写一个多线程程序是不是很难啊？表慌，你马上就会惊诧对于简单情况额外的修改竟然就这么点儿。下面就是使用 `threading` 库改造后的版本了：
```python
import concurrent.futures
import requests
import threading
import time


thread_local = threading.local()


def get_session():
    if not hasattr(thread_local, "session"):
        thread_local.session = requests.Session()
    return thread_local.session


def download_site(url):
    session = get_session()
    with session.get(url) as response:
        print(f"Read {len(response.content)} from {url}")


def download_all_sites(sites):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        executor.map(download_site, sites)


if __name__ == "__main__":
    sites = [
        "https://www.jython.org",
        "http://olympus.realpython.org/dice",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} in {duration} seconds")
```

引入 `threading` 时，程序的整体结构并没有发生改变，只需要做一些小小的调整。`download_all_sites()` 函数从之前的遍历每个站点调用方法变为了一种复杂些的结构。

在这个版本中，我们创建了一个 `ThreadPoolExecutor`，看起来好复杂的样子。我们可以把它的名字拆开来看：`ThreadPoolExecutor = Thread + Pool + Executor`。

之前我们提到过 `Thread`，它可以被看作一个思路。`Pool` 就开始变得有趣了，它可以创建一池子的线程，其中每一个都能并发执行。最后，`Executor` 可以控制池子中的那些线程什么时候以及怎样运行。它将在池子中执行 request 请求。

好在，标准库将 `ThreadPoolExecutor` 实现为上下文管理器，我们可以使用 `with` 语法来管理创建和释放池中的线程。

一旦拥有了 `ThreadPoolExecutor`，我们就可以使用它方便的 `.map()` 方法。这个方法会让 sites 列表中的每一个元素运行传入的函数。最妙的一点在于，它可以控制线程池让其中线程自动并发的执行这些函数。

使用其他语言(包括 Python 2)的同学可能想知道，像类似 `Thread.start()`、`Thread.join()`、`Queue` 这样，我们常用来管理线程细节的方法和对象在 `threading` 库是否还存在。

别担心，都还在，我们仍可以使用它们来实现对线程进行细粒度的控制。而且从 Python 3.2 开始，标准库新增了一种叫做 `Executors` 的更高等级的抽象，如果你不需要太细粒度的控制，可以用它来为你管理很多细节。

例子中另一个有趣的改变是，每一个线程都需要创建自己的 `requests.Session()`。当你看 `requests` 的文档时，可能看不出来是不是需要这样做，但是读过这篇 [issue](https://github.com/psf/requests/issues/2766) 之后会发现，似乎确实每个线程都需要有独立的 Session。

这也是使用 `threading` 的趣味点以及难点，因为操作系统控制着任务们在中断和启动之间来回切换，所以任何在线程间共享的数据都需要保护，或者是保证线程安全。不巧的是，`requests.Session()` 并不是线程安全的。

取决于数据的种类和使用方式的不同，有很多种策略可以保证访问数据时的线程安全。其中一种就是使用线程安全的数据结构，比如 Python 的 `queue` 模块中的 `Queue`。

这些数据结构一般会使用类似 `threading.Lock` 这种低层级的基元来确保在同一时刻只能有一个线程访问同一块代码或同一段内存。`ThreadPoolExecutor` 底层就是以这种策略来实现的，直接用它的对象即可。

例子中也提到了另外一种策略，它被称为线程本地存储。`threading.local()` 创建了一个类似全局变量的对象，但是对于每个线程来说，这个对象又是专门只服务单个线程的。例子中的 `thread_local` 和 `get_session()` 实现了这一点。

```python
thread_local = threading.local()


def get_session():
    if not hasattr(thread_local, "session"):
        thread_local.session = requests.Session()
    return thread_local.session
```

`threading` 模块中的 `local()` 方法专门解决这个问题，虽然看着有些奇怪，但是我们也确实只是需要创建一个对象，而不是为每个线程各创建一个。而且这个对象也会很谨慎地为不同线程隔离不同的数据。

当 `get_session()` 函数被调用时，`thread_local` 会查找到具体是哪个线程正在运行，并为其准备相应的 `session`。每个线程在第一次调用 `get_session()` 时都会创建一个 session，然后在接下来的每次调用中都只会使用这一个 session 对象，始终如一。

最后，稍微提一句线程数量的选择。示例代码中使用了5个线程，你可以随意调整这个数值，然后看看整体时间有什么变化。你可能会想，为每个下载任务都单独设置一个线程应该是最快的，但是并不是如此，至少在我的机器上不是如此。我发现最快记录的线程数大概是在5-10个，而如果数量再增加，线程创建和销毁所付出的额外时间会抵消掉其节省下的时间。

问题的难点就在这里，对于不同的任务来说，最合适的线程数量并不是一个常量，所以经验也是很重要的说。

#### 为什么线程版本如此流行
贼快！这里是我测试的最快的结果，还记得之前非并发版本的耗时么，那可是有14秒之多：
```shell
$ ./io_threading.py
   [most output skipped]
Downloaded 160 in 3.7238826751708984 seconds
```

线程版本的执行时序图如下：

![线程版本执行时序图](https://files.realpython.com/media/Threading.3eef48da829e.png)

它启动了多个线程来同时向多个网站发出请求，使得程序可以将等待时间重叠，以至于可以更快的获取最终结果！流弊！目标达成！

#### 多线程版本的问题

如你所见，为了达到这样的效果，我们稍微增加了一些代码，也确实需要思考一下哪些数据可以在线程间共享。

线程可能会以一些隐秘又难以追踪的方式进行交互，这些交互就很容易造成“竞态条件”(race conditions)，以至于会频繁产生一些随机的、时有时无的，但又很难找到原因的bug。不熟悉竞态条件这个概念想拓展阅读的可以看下面部分。

#### 竞态条件 Race Conditions
竞态条件是对于那些在多线程代码中频繁发生的不易察觉的一类bug的统称。造成竞态条件的原因是，开发人员对于数据访问没有做到充分的保护，没有防止线程间的相互干扰。在编写多线程代码时，你需要做一些额外的操作来确保线程安全。

操作系统会控制你的线程什么时候运行，以及什么时候将它中断然后再换另一个线程开始运行。线程的切换会发生在任何时间点，甚至可能是在执行一段 Python 命令的子步骤时。来看下面这段代码：
```python
import concurrent.futures


counter = 0


def increment_counter(fake_value):
    global counter
    for _ in range(100):
        counter += 1


if __name__ == "__main__":
    fake_data = [x for x in range(5000)]
    counter = 0
    with concurrent.futures.ThreadPoolExecutor(max_workers=5000) as executor:
        executor.map(increment_counter, fake_data)
```

这段代码和之前 `threading` 的例子在结构上非常相似，不同之处在于每个线程都会访问同一个全局变量 `counter`，同时再让它自增。这个变量没有做任何保护，也就是说它不是线程安全的。

为了让 `counter` 自增，每个线程都需要读取它的当前值，然后加一，再将这个值保存回这个变量。这些步骤发生在执行这条语句时：`counter += 1`。

操作系统不会关心代码是怎么写的，它会在任何时刻切换线程执行，这就可能导致有时线程切换发生在该线程读取了 counter 值但没将其自增并写回时，如果新的线程又修改了 counter 的值，那么之前那个线程里却保存着 counter 的历史版本，那问题也就随之出现了。

和你想的一样，出现这种情况的时候非常稀少，可能运行这个程序几千次也看不到一次。这也是这类问题非常难以调试的原因所在，bug相当难复现而且还可能出现随机错误。

在之前的例子中曾提到过，`requests.Session()` 不是线程安全的，这也就意味着如果多个线程使用同一个 session，那么类似上面描述的交互bug就可能会在某些地方出现。提到这一点不是因为我想黑一波 `requests`，而是想指出这些都是难以解决的问题。

### asyncio 异步版本
在开始解释 `asyncio` 示例代码之前，我们先来了解一下 `asyncio` 是如何工作的。

#### asyncio 基础
尽管这里描述的是一个 `asyncio` 的简化版本，有很多细节也没有展示，但是仍能表达出它是如何工作的。

一般来说，我们提到的 `asyncio` 概念时，指的是一个单独的 Python 对象，这个对象被称为事件循环，控制着每个任务何时运行以及如何运行。事件循环了解每一个任务，也知道它们正处于什么状态。真实情况下，任务可能会有很多状态，但是现在，我们可以想象一个简化版的事件循环，他只支持两种任务状态。

就绪状态指，任务有工作需要做，而且可以随时开始的状态；等待状态指，任务正在等待一些外部事物的响应，比如网络操作。

简化版的事件循环会有两个任务列表，一个状态一个。事件循环会挑一个就绪状态的任务让它启动，然后将控制权完全交给这个任务，直到该任务主动将控制权交还给事件循环。

事件循环重新拥有控制权之后，先会把刚刚交还控制权那个任务放入等待状态任务列表中(译者注：这里查阅资料后发现，Python asyncio 中主动通过 await 交还控制权的任务会从 Running 进入 Pending 也就是等待状态，原文 `places that task into either the ready or waiting list` 与此不一致，且后文中提到就绪列表中的所有任务都还没有运行，原文也互相矛盾，故改之)；然后事件循环就会遍历等待状态任务列表，看看它们的 I/O 操作有没有完成。没有遍历就绪列表是因为事件循环知道它们都还没有执行过，也就是说就绪列表还保持着之前就绪的状态。

当所有的任务再一次被分配到正确的列表中之后，事件循环就会挑出下一个任务来运行，简化版事件循环挑选的规则就是，从就绪列表中选一个等了最久的让它运行。整个过程循环往复，至死方休。

`asyncio` 中最重要的一点就是，任务永远不会被剥夺控制权，除非它主动放弃。也就是说，它们肯定不会在执行一个操作的途中被打断。这一点也使得 `asyncio` 在资源共享方面要比 `threading` 稍微容易一些，毕竟开发人员再也不用担心线程是否安全了。

以上，从一个高层级的视角展示了 `asyncio` 的基本理念，如果想对其进行更深入的了解，这篇 [StackOverflow 上的回答](https://stackoverflow.com/a/51116910/6843734) 提供了些很好的细节，相信对于深挖者会有所帮助。

#### async 和 await
现在，我们来讨论一下添加进 Python 中的两个新的关键词：`async` 和 `await`。在上文简要的介绍中，我们看到 `await` 就像有一种魔力，它能让任务主动将控制权交还给事件循环。当代码中使用 await 来标记一个方法调用时，它就代表着这个调用很可能会花费一些时间，任务也应该从这里放弃控制权。

简单来说，`async` 是一个 Python 中的标记，它表明即将要定义的这个函数里会用到 `await`。尽管在某些情况下，这一点并不完全正确(比如 [异步生成器](https://www.python.org/dev/peps/pep-0525/))，但它确实已经 hold 住大部分情况了，在初学者入门时给出一些简单的模型还是绰绰有余的。

下文中也会展示一种例外：代码中使用了 `async with` 语句，这条语句会将一个本该使用 `await` 的对象改为为其创建一个上下文管理器。尽管从语义上看会和之前提到的有些许不同，但本质目的还是一样的：将这个上下文管理器标记为“可以从这里交出控制权”。

可能你会想，管理事件循环和任务之间的交互也太复杂了。其实对于刚开始接触 `asyncio` 的开发者来说，上面提到的那些细节其实没那么重要，但有一点必须要牢记：任何一个需要使用到 `await` 的方法都需要为其标记 `async`，否则会报语法错误。

#### 回到代码
现在你应该对 `asyncio` 有了一些基本的了解，我们可以逐步看看 `asyncio` 版本的示例代码，研究下它是如何工作的。需要注意的是，示例中使用了 [aiohttp](https://aiohttp.readthedocs.io/en/stable/)，运行代码前记得安装一下，`pip install aiohttp`。

```python
import asyncio
import time
import aiohttp


async def download_site(session, url):
    async with session.get(url) as response:
        print("Read {0} from {1}".format(response.content_length, url))


async def download_all_sites(sites):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in sites:
            task = asyncio.ensure_future(download_site(session, url))
            tasks.append(task)
        await asyncio.gather(*tasks, return_exceptions=True)


if __name__ == "__main__":
    sites = [
        "https://www.jython.org",
        "http://olympus.realpython.org/dice",
    ] * 80
    start_time = time.time()
    asyncio.get_event_loop().run_until_complete(download_all_sites(sites))
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} sites in {duration} seconds")
```

这个版本比之前的两个还要再复杂一点点，虽然结构相似，但是启动任务时会比创建 `ThreadPoolExecutor` 再多做一些工作。我们从头看下整个示例。

##### download_site()
最上面的 `download_site()` 函数和 `threading` 版本的几乎完全一致，除了在定义函数前需要用 `async` 标记，以及在调用 `session.get()` 方法时改为使用 `async with` 关键词。后面我们会解释为什么这里的 `Session` 可以直接传递，而不是使用线程本地存储。

##### download_all_site()
你可以在 `download_all_site()` 这个函数中看到与 `threading` 版本相比最大的改动。

session 可以在所有任务中共享，因此这里的 session 被创建为了上下文管理器。任务之间可以共享 session 是因为所有任务都运行在同一个线程上，当 session 处于工作状态时，另一个任务不可能将其中断。

在上下文管理器内，我们使用 `asyncio.ensure_future()` 创建了一个任务列表，同时这个列表还负责运行这些任务。在所有的任务都已经创建完毕之后，我们调用 `asyncio.gather()` 来确保 session 上下文能一直存活，直到所有任务全都完成。

其实 `threading` 版本的代码也做了类似的工作，只不过细节都放权给 `ThreadPoolExecutor` 代为处理了。遗憾的是，截止到目前为止还没有出现类似的 `AsyncioPoolExecutor` 类。

言归正传，代码的细节中其实还埋藏着一个很小但是很重要的改动。还记得之前我们讨论过确定线程创建数量这档子事么？当时我们的结论是 `threading` 版本没办法给出一个确定的数量。

`asyncio` 很酷的一个优势就在于，它规模的伸缩性要比 `threading` 好太多。相比于创建线程，创建异步任务花费的资源和消耗的时长要少的多。因此，创建并且运行更多的任务效果也很好。就像示例中那样，我们为每一个网站都创建了一个独立的任务来执行下载，整体效果也是非常好。

##### \_\_main\_\_
最后，`asyncio` 生态要求我们必须启动一个事件循环，并且告诉它有哪些任务需要运行。示例最下面的 `__main__` 代码块中包含了 `get_event_loop()` 和 `run_until_complete()` 的代码。该说不说的，Python 的核心开发者们为这些函数做的命名确实是不错。

如果你升级到了 Python 3.7，Python 的核心开发者们又简化了这些语法，他们将 `asyncio.get_event_loop().run_until_complete()` 这样的绕口令简化为了 `asyncio.run()`

#### 为什么 asyncio 版本如此流行
真的贼快！在我的机器上测试时，这是最快的一个版本，甚至遥遥领先：
```shell
$ ./io_asyncio.py
   [most output skipped]
Downloaded 160 in 2.5727896690368652 seconds
```

执行时序图看起来和 `threading` 的也非常相似，仅仅使用了一个线程，就将所有的 I/O 请求都完成了：
![asyncio版本执行时序图](https://files.realpython.com/media/Asyncio.31182d3731cf.png)

遗憾的是，由于缺少一个类似 `ThreadPoolExecutor` 这样的装饰器，所以相比与 `threading` 版本在代码上会显得更复杂些。这也意味着要想获得更优秀的性能，我们必须要多做些额外的工作。

还有一个常见的论调是，开发人员得时刻想着在合适的位置添加 `async` 和 `await` 关键词也是一种额外的复杂性。从某种程度上看，这么讲也没毛病。但从反面角度来看，这能强制开发人员思考给定的任务应该在什么时候交出控制权，从而帮助开发人员做出一份更好、更高效的设计。

规模问题也很突出。在上面 `threading` 版本的例子中，为每一个网站分配一个线程所消耗的时间远比只用几个线程要多得多。而 `asyncio` 的例子中有上百个任务，运行起来却一点都没慢。

#### 异步io版本的问题
关于这一点，确实有几个问题需要明确。首先在第三方库方面，如果想要利用好 `asyncio` 的全部优势，就需要依赖特定的异步版本的库。你是否在异步版本的代码中使用 `requests` 库来下载网站，然后发现还是特别慢？这是因为 `requests` 并不支持异步，没办法告知事件循环它正在阻塞状态。但也不用太过担心于此，随着时间的推移，越来越多的第三方库都开始支持 `asyncio` 了，这个问题就会慢慢变得不再是个问题了。

另外一个，也是更不易察觉的问题就是，在协作式多任务处理中，一旦其中有一个任务不配合协作，那么它所有的优势也就都不复存在。代码中任何一个微小的错误都可能造成一个任务长时间的占用控制权不释放，从而导致其他任务被饿死。事件循环主动将其中断也毫无可能，因为任务还没有将控制权交还给事件循环。

考虑到这一点，我们来介绍另一种完全不同的并发方式，`multiprocessing` 多进程。

### 多进程版本
与之前的方法不同，`multiprocessing` 版本的代码可以充分的利用多核处理器的性能。来看下代码：

```python
import requests
import multiprocessing
import time

session = None


def set_global_session():
    global session
    if not session:
        session = requests.Session()


def download_site(url):
    with session.get(url) as response:
        name = multiprocessing.current_process().name
        print(f"{name}:Read {len(response.content)} from {url}")


def download_all_sites(sites):
    with multiprocessing.Pool(initializer=set_global_session) as pool:
        pool.map(download_site, sites)


if __name__ == "__main__":
    sites = [
        "https://www.jython.org",
        "http://olympus.realpython.org/dice",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} in {duration} seconds")
```

这段代码看起来可比 `asyncio` 版本的短多了，而且又和 `threading` 版本非常的相似，但是在我们深挖代码细节之前，首先来了解下 `multiprocessing` 做了些什么。

#### multiprocessing 核心思想
在本小节之前的所有关于并发的示例中，它们都是在电脑中的单个 CPU 或核中运行的，原因和 CPython 当前的设计以及一种叫全局解释器锁(GIL)的东西有关。

本文并不深入讨论 [GIL](/post/37e820d83e914ea38386ad1aa323f3c5/)，我们现在仅需知道前文的同步、线程、异步io版本的代码都只能运行在单核上这一点就足够了。

而标准库中的 `multiprocessing` 库设计的初衷就在于打破这种壁垒，让代码可以在多个 CPU 上运行。从更高层级来看，它会在其他核上创建新的解释器实例，然后把程序的部分代码外包出去交给新的解释器来做。

可以想象，创建一个独立的 Python 解释器不会比在当前解释器创建新的线程要快，这是一项重量级的操作，其间也会有一些限制和困难，但是如果用于解决合适的问题，它就会展示出强劲的动力。

#### multiprocessing 代码


## 如何提升计算密集型程序的速度
### 同步版本
### 多线程异步版本
### 多进程版本
## 什么时候需要使用并发
## 结论