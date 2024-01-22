---
title: "Python虚拟环境构建与新环境部署"
slug: 2a95e2853e784470b1db121a406b90ac
description: 简要记录Python原生虚拟环境的构建和在新环境部署的过程
date: 2024-01-11T10:01:17+08:00
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
---

## 系统环境
本文依赖机器为，64位 `CentOS Linux 7.9.2009`，Python 使用 CentOS yum 直接安装的 `3.6.8` 版本。

**Note:** Windows，MacOS 流程大体相近，具体细节或有出入。Python2不在本文讨论范围。
**Note:** 本文依赖官方pip源，自建pypi的离线模式不在本文讨论范围。
**Note:** 本文虚拟环境仅使用标准库自带的 `venv` 模块，第三方的poetry，pyenv等不在本文讨论范围。
**Note:** 关于项目部署，本文使用直接源码放上去的粗暴方式。打包成可执行文件或者whl安装又或CI/CD等方式不在本文讨论范围。

## 虚拟环境
项目必须使用单独 [虚拟环境](https://zhuanlan.zhihu.com/p/216157886)，项目中用到的所有第三方包都需在虚拟环境中安装。目的是：
1. 项目中安装的依赖不会污染系统环境
2. 项目开发完毕后可以快速整理依赖，方便部署

### 创建
在项目根目录下利用 python 标准库的 `venv` 模块创建名为 `.venv` 的虚拟环境。

``` shell
$ mkdir myproject
$ cd myproject
$ python -m venv .venv
$ ls -la
total 0
drwxrwxr-x. 3 node0 node0 19 Mar 16 15:54 .
drwxrwxr-x. 5 node0 node0 54 Mar 16 15:54 ..
drwxrwxr-x. 5 node0 node0 74 Mar 16 15:54 .venv
```

### 激活
激活后，命令提示符前面会有虚拟环境标识 `(.venv)`。下文操作中带有该标记的即标识在虚拟环境中执行，没有该标记则表示在系统真实环境。
``` shell
$ source .venv/bin/activate
(.venv) $ pip list
Package    Version
---------- -------
pip        21.3.1
setuptools 39.2.0
```

## 开发阶段
**！！！强烈建议！！！** 
还是找一个在线的环境进行开发，离线开发实属给自己找罪受。

### 依赖安装&升级
``` shell
(.venv) $ pip install requests  # 默认安装该包可用的最新版
(.venv) $ pip install requests==21.2  # 安装指定版本
(.venv) $ pip install --upgrade requests  # 升级包到最新版本
```

### [Optional] 升级pip
有时pip版本过低，导致部分第三方包无法安装，可以尝试升级pip版本。
``` shell
(.venv) $ pip list
pip (9.0.3)
setuptools (39.2.0)

(.venv) $ python -m pip install --upgrade pip
(.venv) $ pip list
Package    Version
---------- -------
pip        21.3.1
setuptools 39.2.0
```

## 部署前的准备
当项目开发基本完毕，即将迁移部署测试时，就体现出虚拟环境的优势所在了。
我们可以直接用pip的freeze功能将所在虚拟环境中所有的包及其版本写入一个文件，后续下载依赖安装包进行离线安装，或者直接在线安装都可以以该文件作为指引，方便的一比。

### 导出依赖指示文件
``` shell
(.venv) $ pip list
Package            Version
------------------ ---------
certifi            2022.12.7
charset-normalizer 2.0.12
idna               3.4
pip                21.3.1
requests           2.27.1
setuptools         39.2.0
urllib3            1.26.15

(.venv) $ pip freeze > requirements.txt
(.venv) $ cat requirements.txt
certifi==2022.12.7
charset-normalizer==2.0.12
idna==3.4
requests==2.27.1
urllib3==1.26.15
```

### [Optional] 根据依赖指示文件下载对应安装包到指定目录
如果部署的环境无法联网，则需事先将依赖的所有安装包下载好，部署时一并带到新环境。

``` shell
(.venv) $ pip download -r requirements.txt -d ./packages
(.venv) $ ll packages
total 460
-rw-rw-r--. 1 node0 node0 155255 Mar 16 16:27 certifi-2022.12.7-py3-none-any.whl
-rw-rw-r--. 1 node0 node0  39623 Mar 16 16:27 charset_normalizer-2.0.12-py3-none-any.whl
-rw-rw-r--. 1 node0 node0  61538 Mar 16 16:27 idna-3.4-py3-none-any.whl
-rw-rw-r--. 1 node0 node0  63133 Mar 16 16:27 requests-2.27.1-py2.py3-none-any.whl
-rw-rw-r--. 1 node0 node0 140881 Mar 16 16:27 urllib3-1.26.15-py2.py3-none-any.whl
```


### 项目结构
至此，部署的准备阶段也基本完成，项目的结构基本定型。在省略一些非本文重点讨论的诸如.gitignore，.env或者一些部署自动化shell之后，大概能得到一个类似的项目结构。

``` txt
myproject
  ├── .venv
  │   └─ ...
  ├── packages
  │   ├── certifi-2022.12.7-py3-none-any.whl
  │   ├── charset_normalizer-2.0.12-py3-none-any.whl
  │   ├── idna-3.4-py3-none-any.whl
  │   ├── requests-2.27.1-py2.py3-none-any.whl
  │   └── urllib3-1.26.15-py2.py3-none-any.whl
  ├─ app
  │   ├── __init__.py
  │   ├── config.py
  │   └── main.py
  ├── README.md
  └── requirements.txt
```
可以将其打成一个zip包，方便上传服务器，比如叫 `myproject.zip`

## 开始部署
到了一个新的环境，首先要做到就是将项目压缩包放上去，找个位置解压好后，进到项目目录。然后以前文提到方式创建并激活虚拟环境。

``` shell
$ unzip myproject.zip
$ cd myproject
$ python -m venv .venv
$ source .venv/bin/activate
(.venv) $ pip list
Package    Version
---------- -------
pip        21.3.1
setuptools 39.2.0
```

### 根据依赖指示文件安装依赖
``` shell
(.venv) $ pip install -r requirements.txt  # 在线直接安装
or
(.venv) $ pip install --no-index --find-links=./packages -r ./requirements.txt  # 离线使用安装包安装
(.venv) $ pip list
Package            Version
------------------ ---------
certifi            2022.12.7
charset-normalizer 2.0.12
idna               3.4
pip                21.3.1
requests           2.27.1
setuptools         39.2.0
urllib3            1.26.15
```

### [Optional] 部署脚本
部署中的一些操作其实较为繁琐，在理解其流程后，可将部署过程中项目的环境搭建、启动停止等操作封装为shell脚本，以简化操作。

脚本们可单独放置在一个目录下，比如名为 `scripts`。可根据实际情况编写符合自身项目的脚本，也无固定模式格式要求，主要为了简化操作。

本文计划将脚本分为两个，第一个为环境搭建，包括虚拟环境的构建初始化以及依赖的安装，命名为 `init-env.sh`；第二个为项目服务启停相关，包括状态查询、启动、停止和重启，以项目名称命名 `myproject.sh`。

具体代码可参考 [graft](http://10.10.50.216/lixuechen/graft/tree/master/scripts) 中的脚本编写。
``` shell
$ mkdir scripts
$ cd scripts
$ touch init-env.sh myproject.sh
```

## 参考
Python 官方的包管理平台 [PyPI](https://pypi.org/)
