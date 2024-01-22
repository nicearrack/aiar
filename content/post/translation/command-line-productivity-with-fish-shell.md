---
title: "[译] Fish 命令行的生产效率"
slug: f4273dc3df9447f2ae8f9c58d9df142e
description: 用 Fish 终端处理事务，你将拥有不止一种解决问题的途径，这是她的魅力所在。无止境的定制方案和调整插件可以让事务更快速，更高效的被处理完成。
date: 2020-07-30T09:12:49+08:00
image: https://s11.ax1x.com/2024/01/08/pFShEod.jpg
math: 
license: 
hidden: false
comments: true
categories:
    - 译文
tags:
    - Linux
---

原文链接: [Command line productivity with Fish shell](https://dev.to/yankee/command-line-productivity-with-fish-shell-52e4)

用 Fish 终端处理事务，
你将拥有不止一种解决问题的途径，
这是她的魅力所在。
无止境的定制方案和调整插件可以让事务更快速，
更高效的被处理完成。

在使用 WSL 的时候，
我心爱的 `zsh` 和 `oh-my-zsh` 组合让我陷入了性能危机。
彷徨无助了一个小时后，
我迷迷糊糊的投入了 `Fish Shell` 的怀抱，
听说在 WSL 中，Fish 要比 zsh 快十倍
(不是真实的比例指标，但是赶脚真的非常非常快)。
但是，Fish 也有些她自己的怪癖，后面我会介绍到。

## 安装 Fish Shell

如果你使用的是 `Debian-based` 的发行版：
``` shell
$ sudo apt-get install fish
```
如果你使用的是其他平台，请按照
[这里](https://github.com/fish-shell/fish-shell#getting-fish)
的指示进行操作。

## Fish Shell 介绍

Fish Shell 非常的轻量，反馈表现迅速，并且有着丰富的特性。
这意味着，你只需花费一点精力就能在终端中产生可观的生产效率。
主要特性功能有：

- 语法高亮
- 自动建议命令
- tab 补全
- 方法自动导入

Fish 的官方文档非常丰富和详尽，在本地有一份文档的副本，
你可以通过在 Fish Shell 中键入 `help` 来在浏览器打开这个文档。
对于大多数人来说，安装原生 Fish 就已经足够。
但我们也可以使用插件，主题布拉布拉来使得 Fish 更加可用，和易于调整。
和其他 Shell 类似，Fish 有着大量的插件安装框架，
而我们将使用的插件安装框架被称为 `oh-my-fish(omf)`。

## omf 介绍

omf 是在 Fish Shell 之外，最上面薄薄的一个层，
因此你不必担心速度和性能的问题。可以通过一条简单的命令安装它：
``` shell
$ curl -L https://get.oh-my.fish | fish
```
安装完毕后，你的 Fish Shell 将获得一个 `omf` 命令，
这个命令可以安装主题和其他有用的插件。omf 十分直观，
如果你曾使用过 nvm 或者 pip，你会有种似曾相识的感觉。

### omf 主题

omf 有着各种各样的主题供你选择。你可以找到托管在 omf 的所有主题，
或者你可以使用 `omf theme` 命令来列出所有可用主题，
已安装主题和默认主题。

安装新主题时，当前 Fish Shell 客户端将会直接应用该主题。
``` shell
$ omf install <theme-name>
$ omf install bira
```
![展示主题-实例](https://res.cloudinary.com/practicaldev/image/fetch/s--_3T8aDWd--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://cdn-images-1.medium.com/max/2032/1%2ARFN2ONxk2-Lzn_K9uUptzg.gif)
_列出主题和切换主题_

如果你拥有很多主题，可以这样来在它们之间切换：
``` shell
$ omf theme <theme-name>
```

### omf 别名

当你需要完成一些重复的任务时，
omf 的别名功能可以让你尽可能的减少键盘敲击次数。
Fish Shell 的 `alias` 命令可以用来定义操作的别名。
你可以通过命令行轻易的使用它。
``` shell
$ alias <alias> '<command>' -s
$ alias install 'sudo apt-get install' -s
$ alias remove 'sudo apt-get remove --purge' -s
```
定义了以上的别名后，你就可以通过 `install` 命令来安装所有的包，
或者使用 `remove` 命令来完全删除某个包。
``` shell
$ install vim
$ remove python2.7
```
![创建别名](https://res.cloudinary.com/practicaldev/image/fetch/s--YLN2JNx---/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://cdn-images-1.medium.com/max/2032/1%2AWThV-osTr7qeVxtDH7cSiw.gif)
_创建别名install_

通过 `alias` 命令创建的别名可以一直保持到本次会话结束，
也就意味着，如果你打开了一个新的终端进程，之前创建的别名将不再工作。

如果想让之前定义的别名可以在新的终端工作，我们需要使用 `-s` 标记。
它将在后台使用 `funcsave`。

使用了 `-s` 标记的别名定义，将被定义为永久性的别名，
本机的任何 Fish 终端都可以使用。

当你忘记了自己定义过啥别名，
你可以使用 `alias` 命令来浏览所有已定义的别名。
![列出所有别名](https://res.cloudinary.com/practicaldev/image/fetch/s--D4RMiEnz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://cdn-images-1.medium.com/max/2032/1%2AvQumglI-6OpHawA-I5L9nA.gif)
_列出所有别名_

### 使用 nvm

Fish Shell 的其中一个怪癖是，他不能像 nvm 一样运行 bash 工具。
为此，你需要一个叫做 [bass](https://github.com/edc/bass)的包，
来将 nvm 暴露给 Fish Shell。

bass 创建了一个可以支持其他 bash 工具包的框架，
比如我们稍后会使用一个叫 `fish-nvm` 的工具包。
也有很多其他用于 nvm 的工具包，但是 fish-nvm 不会影响性能。
``` shell
$ omf install bass
$ omf install https://github.com/FabioAntunes/fish-nvm
```

#### 使用 virtualenv

这是使用 Fish Shell 的一个陷阱。当你在 Python 的虚拟环境工作时，
你 *不能* 使用 下面这样普通的方式激活虚拟环境：
``` shell
$ python -m venv venv
$ source venv/bin/activate
```
而是要使用下面的方式：
``` shell
$ source venv/bin/activate.fish
```

## 高效的包

### pj

(译者注：翻译的挺烂的，原文写的也不咋地，主要看图和gif大概就能懂)

pj 会以一种预测的方式，在你喜欢的目录之间跳转。
告诉 pj 去哪里寻找你的项目或者文件夹，
然后他可以通过 tab 来补全。
``` shell
$ omf install pj
```
比如，在 `home`  目录下有个 `test` 文件夹，里面有一堆别的文件夹。
![hhh](https://res.cloudinary.com/practicaldev/image/fetch/s--Sv_aZ2Gb--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://cdn-images-1.medium.com/max/2000/1%2AWGRQj64vuEugFL_z2Ffzfw.png)
为了将 `test` 文件夹标记为跳转目标，我们需要设置这样的项目地址：
``` shell
$ set -Ux PROJECT_PATHS ~/test
```
现在，我们可以在任意位置来访问 test 内的文件夹了。
![pj jump](https://res.cloudinary.com/practicaldev/image/fetch/s--bSfr11nc--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://cdn-images-1.medium.com/max/2272/1%2AjI0uddiur0aJXbsjmz1y_g.gif)
_pj 的作用_

### z

z 和 pj 有些相似，但从某种意义上来说 z 更加智能，
它会持续跟踪你最常访问的一些文件夹，因此你可以轻松的跳到这些位置。
``` shell
$ omf install z
```
就像我说的，z 是一个智能的工具，即使我输入了错别字，
它也会从我的最常访问里努力匹配到与输入最相近的那一个。
![z](https://res.cloudinary.com/practicaldev/image/fetch/s--sxD8x3zV--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://cdn-images-1.medium.com/max/2000/1%2Ax8va4Ph_V_ADbMSre0PasA.gif)
_z 的作用_

### plugin-git
和 zsh 中的 git 插件类似，plugin-git 包会给予你一个标准 git 别名集合
来加速你的 git 工作流。
``` shell
$ omf install https://github.com/jhillyerd/plugin-git
```
![plugin-git 的作用](https://res.cloudinary.com/practicaldev/image/fetch/s--vSUKKMXZ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://cdn-images-1.medium.com/max/2000/1%2AN9xOm9M149wtYDQdlT7B3w.gif)
_plugin-git 的作用_

不只这样，为了确保你使用正确的别名，
它也会将别名进行展开，来形成完整的命令。
这里有完整的[别名列表](https://github.com/jhillyerd/plugin-git#usage)。

### fzf
Fuzzy Finder(模糊查找) 或者 fzf 是一种更加快速的通用查找工具，
可以用它来查找文件或者命令历史。
``` shell
$ omf install https://github.com/jethrokuan/fzf
```
搜索遍历你的命令历史记录，你可以使用 `ctrl + r` 或者输入该命令的某些部分，
然后敲击 `ctrl + r` 来精准查找符合条件的命令。
![fzf 的使用](https://res.cloudinary.com/practicaldev/image/fetch/s--zbqXU3cs--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://cdn-images-1.medium.com/max/2000/1%2AlOEP-GYgbG07VnTca6o18g.gif)
_fzf 的使用_

如果你想在当前目录下搜索文件，你可以使用 `ctrl + o` 然后浏览他们。
你可以用这个工具做更多事情，点击[这里](https://github.com/jethrokuan/fzf#usage)查看更多。

## 总结
我希望这篇文章能很好的指导安装 Fish 和提升工作流的效率。如果你有任何建议或问题，
下面评论就好~
