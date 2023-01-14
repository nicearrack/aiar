---
title: "[译] 什么是AUR，如何在Arch和Manjaro中使用AUR"
description: 如果你用过 Arch Linux 或者其他基于 Arch 的 Linux 发行版(比如 Manjaro)，那么可能曾遇到过 `AUR` 这个术语。在你尝试安装一个软件的时候，也许有人会建议你从 `AUR` 来安装它，然后你满脸问号。什么是 `AUR`？为啥要用它？又要咋用？接下来，我将回答这些问题。
date: 2021-05-21T09:01:54+08:00
image: https://raw.githubusercontent.com/Arrackisarookie/images/main/aur.jpg
math: 
license: 
hidden: false
comments: true
categories:
    - 译文
tags:
    - Linux
---

> 原文: [What is Arch User Repository (AUR)? How to Use AUR on Arch and Manjaro Linux?](https://itsfoss.com/aur-arch-linux/)

> 最近更新: 2020/9/18 - [Dimitrios Savvopoulos](https://itsfoss.com/author/dimitrios/)

如果你用过 [Arch Linux](https://www.archlinux.org/) 或者其他 [基于 Arch 的 Linux 发行版](https://itsfoss.com/arch-based-linux-distros/)(比如 Manjaro)，那么可能曾遇到过 `AUR` 这个术语。在你尝试安装一个软件的时候，也许有人会建议你从 `AUR` 来安装它，然后你满脸问号。

什么是 `AUR`？为啥要用它？又要咋用？接下来，我将回答这些问题。

![whatisaur](https://i1.wp.com/itsfoss.com/wp-content/uploads/2020/04/what-is-aur.png?w=800&ssl=1)

## 什么是 AUR
`AUR` 是 `Arch User Repository` (Arch 用户资料库) 的缩写。这是一个
社区驱动的资料库，它专注于服务那些使用 Arch 或者 Arch-base Linux 发行版的用户。AUR 包含一个名叫 [PKGBULIDs](https://wiki.archlinux.org/index.php/PKGBUILD) 的软件包说明(package descriptions)，它可以让你通过 [makepkg](https://wiki.archlinux.org/index.php/Makepkg) 来编译你所需的软件包，然后使用 [pacman](https://wiki.archlinux.org/index.php/Pacman#Additional_commands)(Arch Linux 的软件包管理器)来安装这个软件包。

`AUR` 的创立是为了组织和分享那些来自社区的新软件包，以便于加速受欢迎的软件包被纳入[社区资料库](https://wiki.archlinux.org/index.php/Community_repository)。

有相当一部分进入官方资料库的新软件包开始时都是来源于 AUR。在 AUR，用户可以建设他们自己的软件包建设器(package builds，PKGBUILD 和相关文件)。

AUR 社区支持为社区中的软件包投票，如果某一个包足够受欢迎(假设它有兼容的许可证和良好的包装技术)，那么它可能可以直接进入社区资料库，这样就可以直接由 pacman 直接访问了。

> 简而言之，AUR 就是一种特殊的软件安装途径，它允许开发人员在软件未正式纳入 Arch 资料库时，将该软件提供给 Arch Linux 用户以供安装使用。

## 应该使用 AUR 么？有什么风险？
使用 `AUR` 就像横穿马路，如果小心谨慎，那就没什么问题。

如果你是一个 Linux 新手，建议还是不要轻易使用 `AUR`，直到你已经全面的了解了 `Arch/Manjaro` 和 `Linux` 的基础知识。(译者注: 个人感觉 AUR 类似 GitHub)

的确，任何人都可以向 AUR 上传软件包，但是 [Trusted Users](https://wiki.archlinux.org/index.php/Trusted_Users)(TUs) 会负责密切关注上传的内容。尽管 TUs 会对上传的文件进行质量管控，但是仍无法保证 AUR 中的软件包格式正确或者无危胁。

在实践过程中，AUR 看起来仿佛很安全，但是理论上它是可以造成一些损害的，当然，这只会发生在你疏忽大意的时候。毕竟，从 AUR 安装软件时，机智的 Arch 使用者们应该每次都会检查 `PKGBUILDs` 和 `*.install` 文件。

另外，如果 AUR 中的软件包被纳入了 core/extra/community，那么 TUs(Trusted Users)会将 AUR 中删除该包，因此它们之间不应存在命名冲突。AUR 中经常包含着开发中的软件包(cvs/svn/git/etc)，他们将会被重命名，比如 foo-git。

对于 AUR 中的软件包，`pacman` 会处理其依赖关系并检测文件冲突，因此不必担心某个包中的文件会将另外一个包的文件覆盖掉。除非你在默认情况里添加了 `-force` 选项，如果你真这么做了，可能会遇到一堆比文件冲突更严重的问题。

## 如何使用 AUR
使用 AUR 最简单的方法就是通过一个 `AUR helper`。大部分的 [AUR helper](https://itsfoss.com/best-aur-helpers/) 是命令行工具，有些也支持 GUI 图形化操作。这种工具支持搜索和安装那些发布在 AUR 上的软件包。

### 在 Arch Linux 上安装一个 AUR helper
假设你想使用的是 [Yay AUR helper](https://github.com/Jguer/yay) 这款工具。首先需要确保你的 Linux 上已经安装了 `git`。然后 `clone` 这个仓库，随后进入该文件夹，最后执行安装。

``` shell
$ sudo pacman -S git
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
$ makepkg -si
```

安装完毕后，就可以像下面这样使用 `yay` 命令安装软件了~

``` shell
$ yay -S package_name
```

当然，也不是说只有使用 AUR helper 才可以从 AUR 安装软件，下一部分你将看到如何不用 AUR helper 使用 AUR。

### 不用 AUR helper 安装 AUR 软件包
如果你不想使用 AUR helper，你也可以自行从 AUR 安装软件包。

建议，在 [AUR page](https://aur.archlinux.org/) 中找到想安装的软件包之后，请参照该软件的 `Licence`，`Popularity`，`Last Updated`，`Dependencies` 等指标，判断其内容质量。

``` shell
$ git clone [package URL]
$ cd [package name]
$ makepkg -si
```

假设，你想安装 [telegram desktop](https://aur.archlinux.org/packages/telegram-desktop-git) 这个软件：

``` shell
$ git clone https://aur.archlinux.org/telegram-desktop-git.git
$ cd telegram-desktop-git
$ makepkg -si
```

## 在 Manjaro Linux 中启用 AUR 支持
Manjaro Linux 默认不启用 AUR，需要使用 `pamac` 来启用这一功能，我的笔记本使用的是 [Manjaro](https://manjaro.org/) Cinnamon，但下面的步骤适用于所有版本的 Manjaro。

打开 Pamac 也就是 `Add/Remove Software`：
![打开Pamac](https://i1.wp.com/i.imgur.com/kFF6HtW.png?ssl=1)

打开 Pamac 之后，进入属性 `Preferences`：
![打开属性](https://i0.wp.com/i.imgur.com/47r963A.png?ssl=1)

打开属性对话框，进入 AUR 页签，启用 AUR 支持，启用检测更新，然后关闭对话框。

![启用AUR](https://i1.wp.com/i.imgur.com/UThiDHO.png?ssl=1)

现在，搜索软件的时候可以搜到源自 AUR 的软件包了，可以通过软件描述下面的标签来区分。

![区分源自AUR的软件](https://i2.wp.com/i.imgur.com/RM5BKi2.png?ssl=1)

AUR 是大家 [热爱 Arch Linux 的众多原因](https://itsfoss.com/why-arch-linux/) 之一，你可以看到它为什么如此流行。

希望本文对你有用。

希望能看到各大社交媒体上即将出现的 Arch 主题~
