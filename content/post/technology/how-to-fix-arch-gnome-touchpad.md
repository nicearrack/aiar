---
title: "Arch+Gnome 触摸板功能忽然不全了怎么办"
slug: e93e60bf2eca473da47d60de30ebcdba
description: 某一次重启之后, 触摸板无法 tap to click, 无法触发三指四指的手势事件, 一指两指滑动功能还可以使用。以下记录修复过程。

date: 2022-02-21T17:22:12+08:00
image: https://s11.ax1x.com/2024/01/08/pFSgY7R.jpg
math: 
license: 
hidden: false
comments: true
categories:
    - 技术
tags:
    - Linux
---


> 万物皆有裂痕，那是光照进来的方向。



## 问题描述
某一次重启之后, 触摸板无法 tap to click, 无法触发三指四指的手势事件, 一指两指滑动功能还可以使用。

gnome-control-center touchpad 设置消失, gnome-tweek 设置 touchpad 无效, dconf-editor 设置 touchpad 无效。

重启之前的操作可能与触摸板相关的有:
1. 安装了 todesk 远程控制软件，可能会重载触摸板的配置
2. sudo pacman -Syu

目前只想到这两个, 不过确实很久没有关机了, 大概有一周左右, 期间滚动升级了很多次, 不过都没有重启或关机

## 猜测原因
滚动更新可能触摸板依赖默认安装了 synatics 导致和 gnome 不兼容, [Arch Wiki](https://wiki.archlinux.org/title/GNOME#Mouse_and_touchpad) 里这样写着:

> Note: The synaptics driver is not supported by GNOME. Instead, you should use libinput. See this [bug report](https://bugzilla.gnome.org/show_bug.cgi?id=764257#c12).

或者也可能是由于 [todesk](https://www.todesk.com/linux.html) 重载触摸板配置啥的我也不太清楚

## 解决方案

### 前置条件
1. 默认系统已经安装类似 yay 的 AUR 包管理
2. 默认桌面系统是Gnome, 且已可以安装插件

### 具体操作

1. remove xf86-input-synaptics package
```
$ sudo pacman -R xf86-input-synaptics
```

2. install xf86-input-libinput
```
$ sudo pacman -Syu xf86-input-libinput
```

3. reboot
```
$ reboot
```

重启之后大概是可以触发 touch to click 了, 如果三指手势仍无法触发, 则进行下面几步

4. install touchegg
```
$ yay -Syu touchegg
```

5. enable touchegg
```
$ sudo systemctl enable --now touchegg
```

6. install gnome extension x11-gestures  
   安装该插件即可  
   https://extensions.gnome.org/extension/4033/x11-gestures/

7. 一般来说装好插件就三指手势已经可以触发了, 不行的话可以重启试试看



## 参考文档
https://bbs.archlinux.org/viewtopic.php?pid=1638596#p1638596  
https://itsfoss.com/three-finger-swipe-gnome/  
https://wiki.archlinux.org/title/GNOME#Mouse_and_touchpad  
https://bugzilla.gnome.org/show_bug.cgi?id=764257#c12  
