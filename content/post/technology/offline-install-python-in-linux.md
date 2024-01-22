---
title: "在Linux中离线编译安装Python"
slug: 0820636dfb6845c58164439d206e9baf
description: 最近在将 Python3.6 升级为 Python3.9，记录下过程。
date: 2024-01-11T10:07:40+08:00
image: https://s11.ax1x.com/2024/01/08/pFShAdH.png
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - 技术
tags:
    - Python
    - Linux
---

## 概述
最近在将 Python3.6 升级为 Python3.9，记录下过程。
由于服务器环境均无法连接互联网，故采用从可联网机器上下载安装包再上传至服务器离线编译安装的方式。

本文默认没有专职服务器运维人员，个人拥有服务器所有权限。（有专业人员哪还用得着自己装。。ORZ）

下文在 `CentOS 7.8` 中实测成功。

## 依赖
在安装 Python 时，程序会依赖一些系统程序。为了能正确安装，需要提前将这些依赖装好。

可以在 [这里](https://pkgs.org/) 按照系统版本和所需包搜索下载。

ps: 说实话，其实装的时候还是有点慌的

*以 CentOS 7.8 为例，其他系统版本需编译试错、搜索*
**编译工具** (使用的是系统自带的)
1. gcc == 4.8.5
2. make == 3.82

**系统依赖**
1. zlib-devel  # 影响 zlib 包的安装
2. libffi-devel  # 忘了影响啥，装就对了
3. openssl-devel  # 影响 \_ssl 包的安装，如果装了这个还是无法正确安装，则可以看 [这篇文章(还没写)]()
4. bzip2-devel  # 影响 \_bz2 包的安装

## 下载
Python官方下载地址：https://www.python.org/downloads/
Python官方 ftp 下载地址：https://www.python.org/ftp/python/

可以按需下载相应的 Python 版本，由于是要在 Linux 中编译安装，所以我们需要下载 `Gzipped source tarball` 或者 `XZ compressed source tarball`，前者是 `.tgz` 格式，后者是 `.tar.xz`，正确解压后的结果无本质区别。

每个大版本中较新的版本建议从 ftp 中下载，downloads 中不是很全。

需要注意的是，Python 的同一个大版本在绝大多数情况下对于第三方包来说区别不大，
即 `Python 3.9.*` 中所有的版本，绝大部分情况都可以通用，差别细节详见官网版本介绍。

## 解压
下载之后我们就会得到一个类似 `Python-3.9.13.tgz` 这样的文件，将其上传到服务器之后，随便在一个地方进行解压即可，这个文件可以理解为一个安装包，具体安装到哪里需要后续配置。

``` shell
$ cd /path/to/xxx
$ tar -xvf Python-3.9.13.tgz
```

## 配置
然后进入到解压出的文件夹中，开始配置安装规则。

常用的参数就是 `--prefix` 这个参数，它决定了Python将被安装在哪里。
如果不填，将会默认安装到 `/usr/local/python` 中，如此则后面的编译安装命令需要root权限，但一般 **不建议** 这样，因为如此可能会导致将操作系统之前已有的 Python 强制覆盖，引发一系列不可控的问题。

该示例如果完成安装，则 Python 命令的可执行文件位置是 `/home/app/depends/python/py39/bin/python3.9`

``` shell
$ cd Python-3.9.13/
$ ./configure --prefix=/home/app/depends/python/py39
```

配置成功后，可能会有类似下文这样的建议，但笔者亲测，当 gcc 版本比较低时(比如本例中的4.8.5)，使用该建议无法编译成功。所以建议低版本的 gcc 不要使用 `--enable-optimizations` 参数。

```
If you want a release build with all stable optimizations active (PGO, etc),
please run ./configure --enable-optimizations
```


## 编译
配置如果没有问题，就可以开始编译了，如果安装到了系统目录下则需要root权限。

``` shell
$ make
```

如果依赖没有安装，或者遗漏，那么配置、编译这两步可能需要反复几次。
判断自己需要的内置包能不能正常安装，就需要看 make 的执行结果，最后下面一块的内容会有类似下面这样的输出。如果其中没有 `Failed` 或者 `error` 等关键词出现，并且 `necessary` 中缺少的包你未来也用不到，那么恭喜你可以进行下一步 **安装** 了。

```
Python build finished successfully!
The necessary bits to build these optional modules were not found:
_tkinter
To find the necessary bits, look in setup.py in detect_modules() for the module's name.


The following modules found by detect_modules() in setup.py, have been
built by the Makefile instead, as configured by the Setup files:
_abc                  atexit                pwd
time
```

示例中的这次编译意味着，`_tkinter` 由于缺少相应的依赖，将不会被安装。搜索后得知，缺少 `tkinter` 这个包，可以参照上文 [依赖](#依赖) 进行安装。

具体哪些包依赖什么，可以以这样的关键词搜索: `[系统版本] [Python版本] [缺失的包名] not found`，以 `_tkinter` 为例，即 `CentOS7 Python3 _tkinter not found`。
如果是国外或开源的操作系统(RedHat、CentOS、Arch、Ubuntu等)建议使用 Google，无法科学上网则建议使用 bing 国际版。如果是国内自主开发的系统，建议使用bing，且尽可能看官方建议。

拒绝百度，从我做起。

## 安装
一般来说，编译能过，而且不缺依赖，则安装就会比较顺。所以，请尽可能确保编译无误后，最后再执行安装。

``` shell
$ make install
```

没报错就算成功，如果配置时用了 `--prefix` 参数指定了一个 **不在** `$PATH` 中的地址，则会有下面这样的提示。
意思是，没办法直接用 `python` 命令直接执行刚刚安装好的程序。

```
Installing collected packages: setuptools, pip
  WARNING: The scripts pip3 and pip3.9 are installed in '/path/to/python/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed pip-22.0.4 setuptools-58.1.0
```

查看 PATH 命令
``` shell
$ echo $PATH
```

如果需要在终端直接执行 `python`，这里提供两种方法来实现。

1. 使用软连接
可以使用软连接将刚刚安装的 Python 放到 `$PATH` 中的某个路径下，类似创建了一个快捷方式。
比如，安装目录为 `/new/path/to/python`，PATH 中有的路径 `/xxx/xxx`
``` shell
$ ln -s /new/path/to/python/bin/python3.9 /xxx/xxx/python3.9
$ ln -s /new/path/to/python/bin/pip.9 /xxx/xxx/pip3.9
```
2. 直接将安装目录放到 PATH 中
假设默认 shell 是 bash，安装目录为 `/new/path/to/python`。则修改该用户的 .bashrc 文件即可。
``` shell
$ vim ~/.bashrc
~/.bashrc
export PATH=/new/path/to/python/bin:$PATH
```
保存文件后，执行
``` shell
$ source ~/.bashrc
```
以后该用户就可以直接使用 `python3.9` 命令了。
