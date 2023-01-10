---
title: "[译] 相见恨晚的 Git 命令"
description: 我辈开发者使用最多的技术既不是 JavaScript，也不是 Python 或者 HTML。它甚至在面试中都很少被提到，也很少被列入工作的必备技术栈。没错，我说的正是 Git 和版本控制。
date: 2020-07-03T08:54:48+08:00
image: 
math: 
license: 
hidden: false
comments: true
categories:
    - 译文
tags:
    - Git
---

> - 原文地址：[Git Concepts I Wish I Knew Years Ago](https://dev.to/g_abud/advanced-git-reference-1o9j)
> - 原文作者：[Gabriel Abud](https://dev.to/g_abud)
> - 译文出自：[DEV Community](https://dev.to/)
> - 译者：[Arrackisarookie](https://github.com/Arrackisarookie)

我辈开发者使用最多的技术既不是 JavaScript，
也不是 Python 或者 HTML。
它甚至在面试中都很少被提到，也很少被列入工作的必备技术栈。

没错，我说的正是 Git 和版本控制。

长久以来，我们大部分开发人员只学过一点点 Git 的概念。
这些知识仅仅能让我们拥有在一个小团队内使用简单功能分支工作流的能力。
如果你也像我一样，这种状态将会伴随你的职业生涯很久。

是时候再访 Git，重新审视一下掌握它对我们职业生涯的提升有多么重要。
本指南可以作为一篇参考，它包含了一些我认为很是重要但可能鲜为人知的概念。
掌握 Git 之后，你管理代码的方式以及每天的工作流将会发生巨大的改变。
由于 Git 命令有些陈旧并且难以记忆，因此本文将会按照概念和预期表现进行分解。

如果你对 Git 的基本概念掌握的不够牢固，比如工作目录、本地仓库和远程仓库之间的区别，
那么建议你可以先阅读[这篇指南](https://dev.to/unseenwizzard/learn-git-concepts-not-commands-4gjc)。
同样，如果没有掌握 Git 的基本命令，可以从[官方文档](https://git-scm.com/docs/gittutorial)开始学习。
本文并不意味着会带你从一个彻底的新手变成专业人员，
而是默认你已经熟练掌握如何使用 Git。

## 基本 Git 命令

### 日志

#### 我刚干哈了
``` shell
$ git log
$ git log --oneline  # 更为精炼的输出
$ git log --graph  #  以分支的可视化图显示
```

#### 查看你的撤销历史
``` shell
$ git reflag
```
因为有时 `git log` 命令无法捕捉到撤销的命令，
特别是对于那些无法在 commit 历史里显示的命令。

在你运行了类似 `git rebase` 这样的“危害型”命令后，
`reflag`  基本上可以算是一层安全网。
你不仅可以看到之前所做的 commit，
而且还将看到导致 commit 的每一个过程。
看这篇 [Atlassian 上的文章](https://www.atlassian.com/git/tutorials/refs-and-the-reflog)
来了解更多关于 refs 运作方式。

#### 查看当前状态 + 任何合并冲突
``` shell
$ git status
```
虽然 `git status` 是一个非常基础的命令，我们很早之前就学过，
但是，由于它作为 Git 内部基本原理的学习工具，其重要程度仍然值得
我们重复学习。
它还可以帮助你浏览复杂的 rebase 和 merge 过程。

#### 对比 staged (或者 unstaged) 中的异同
``` shell
$ git diff --staged  # staged 的改变
$ git diff   # unstaged 的改变
```

#### 对比两个分支之间的异同
``` shell
$ git diff branch1..branch2
```

### 导航 (Navigation)
#### 我想看看我之前干哈了
``` shell
$ git reset <commit-sha>
```
这条命令将会撤销对应 commit，并且取消那次 commit 中的 stage 操作，
但是那些文件仍然保留在工作目录。

#### 我想切换到别的分支
``` shell
$ git switch branch-name  # Git 2.23 中的新语法
$ git checkout branch-name  # 经典语法
```
`git checkout` 可能会有些让人难以理解，因为他既可以工作在文件层级，
也可以工作在分支层级。
从 [Git 2.23](https://www.infoq.com/news/2019/08/git-2-23-switch-restore/)
开始，我们拥有了两个新的命令：

- `git restore` 用来 checkout 文件
- `git switch` 用来 checkout 分支

(译者注：详细可访问 [官方文档](https://git-scm.com/docs/git-restore)
和 [Stack Overflow 相关提问](https://stackoverflow.com/questions/58003030/what-is-git-restore-command-what-is-the-different-between-git-restore-and-git))

如果你想避免 `git checkout` 造成的困扰，上面两个命令非常适合你。

#### 我想回到之前我在的分支
``` shell
$ git switch -
```

### 修改 (Modifications)
#### 我把自己挖进了兔子洞，让我们重新开始
(译者注: get 不到这个梗。。)

``` shell
$ git reset --hard HEAD
```
这条命令将重置本地目录到最近一次 commit 的状态，并且会放弃所有 unstage 的文件。

#### 我想把一个文件重置到之前的样子
``` shell
$ git restore <filename>  # Git 2.23 新语法
$ git checkout -- <filename>  # 经典语法
```
#### 我想撤销上一次 commit 并且重写历史
``` shell
$ git reset --hard HEAD~1
```
#### 我想回到 n 次 commit 之前
``` shell
$ git reset --hard HEAD~n  # 回到倒数第 n 次 commit
$ git reset --hard <commit-sha>  # 或者回到特定的某次提交
```
`soft`，`mixed` 和 `hard` 三种 reset 的不同：

- `--soft`：撤销 commit 但是工作目录中会保留更改
- `--mixed` (默认)：撤销 commit，撤销当次 commit 的 stage，但是工作目录中会保留更改
- `--hard`：撤销 commit，撤销当次 commit 的 stage，并且删除更改

#### 我已经重写了历史记录，现在想把这些改变 push 到远程仓库
``` shell
$ git push -f
```
只要你的本地仓库和远程仓库有差异，这一步都是必要的。

*WARNING*：强制 push 需要格外小心。一般来说，
在共享的分支你应该避免任何的强制 push。在开启一个 pull 请求之前，
你应将强制 push 限制在你自己的分支内，以免在不经意间弄乱你队友的 git 历史。

#### 我想为上一次 commit 多加一些改变
``` shell
$ git commit --amend
```

#### 我想重写本地的一堆 commit
``` shell
$ git rebase -i <commit hash>  # commit hash 是所有你想改变的 commit 之前的一个 commit 的 hash 
```
这条命令将开启一个互动提示，你可以通过它来选择保持、压缩、删除
哪一个 commit。你也可以在这里改变 commit message。
比如在清理错字或者规范化 commit 时，它非常有用。

当深入学习 Git 之后，我发现 rebasing 是非常令人困惑的主题之一。
查看这个 [rebasing](https://dev.to/g_abud/advanced-git-reference-1o9j#rebase-vs-merge)
文档了解更多。

#### 这个 rebase 垮了，报废掉它吧
``` shell
$ git rebase --abort
```
你可以在 rebase 过程中使用这条命令。

我发现，rebase 带来的麻烦总是超过他的价值，特别是在 rebase 两个
有着大量相同更改的分支的时候。在完成整个 rebase 之前，
你都可以让这个 rebase 流产。

#### 我想从另一个分支把一个 commit 带到当前分支
``` shell
# 将 commit-sha 所指的 commit 带入当前分支
$ git cherry-pick <commit-sha>
```

#### 我想从另一个分支把一个指定的文件带到当前分支
``` shell
$ git checkout <branch-name> <filename>
```
(译者注：这个仿佛不能用 git restore，git restore 更倾向于重置和恢复)

#### 我想在版本控制中停止追踪某个文件
``` shell
$ git rm --cached <file-name>
```

#### 我需要更换分支，但当前状态已有更改
``` shell
$ git stash  # 将已有更改保存在 stash 栈的栈顶
$ git stash save "对于更改的信息描述"
$ git stash -u  # 同时也 stash 未被追踪 (untracked) 的文件
```

#### 我想看看我的 stash 里有啥
``` shell
$ git stash list
```

#### 我想把 stash 里的东西取出来
``` shell
$ git stash pop  # 弹出最近添加到 stash 栈的项目
$ git stash apply stash@{stash_index}  # 申请取出指定项目可以用 git stash list 查看
```

#### 我想撤销一次 commit 而不重写历史
``` shell
$ git revert HEAD  # 撤销最近一次 commit
$ git revert <commit-sha>  # 撤销指定 commit
```
这条命令将重新运行提交新的 commit 时的逆过程，
从而撤销你的更改而不会撤销历史记录。
在共享的分支中撤销 commit 时，重写历史记录会非常的复杂，
所以使用 git revert 是一种很安全的解决方式。

### 清理 (Cleanup)
#### 我去，咋有这么多分支
``` shell
$ git branch --no-color --merged | command grep -vE "^(\+|\*|\s*(master|develop|dev)\s*$)" | command xargs -n 1 git branch -d
```
这条命令将删除本地除了 master、develop、dev 之外的所有已合并的分支，
如果你的主分支和 dev 分支有着另外的名字，
你可以改变相应的 `grep` 的正则。

这条命令很长，不太好记，但你可以为它设置一个别名，就像这样：
``` shell
$ alias gbda='git branch --no-color --merged | command grep -vE "^(\+|\*|\s*(master|develop|dev)\s*$)" | command xargs -n 1 git branch -d'
```
如果你在使用 `Oh My Zsh` ，这一步它已经为你完成。
查看 [aliases](https://dev.to/g_abud/advanced-git-reference-1o9j#aliases)
了解更多。

#### 来清理旧分支和无效 commit 吧~
``` shell
$ git fetch --all --prune
```
如果你已经为远程仓库设置了在 merge 时删除分支，这条命令也非常有用。

(译者注：git fetch --prune 将会在 fetch 前
移除在本地的所有远程仓库中不再存在的远程跟踪引用，详见[官方文档](https://git-scm.com/docs/git-fetch#Documentation/git-fetch.txt---prune))

### Aliases
Git 命令有时会很长，不太容易记住。我们不想每次都把它们敲一遍或者
花费几天将它们背下来，因此我强烈建议你为它们设置 Git 别名。

更方便的方式是，安装一个像 `Z Shell` (Zsh) 中的 `Oh My Zsh` 一样的工具。
这样一来，你将拥有[一大堆最常用的 Git 命令的别名](https://github.com/ohmyzsh/ohmyzsh/wiki/Cheatsheet#git)。
你可以使用 `别名 + tab` 来补全他们。我懒得按照我的喜好设置 shell，
所以我喜欢用一些类似 Oh My Zsh 的开源工具，它们可以帮我配置好~
更不用说它们还有这漂亮的外观了~

我每天用的最多的一些命令：
``` shell
$ gst - git status
$ gc  - git commit
$ gaa - git add --all
$ gco - git checkout
$ gp  - git push
$ gl  - git pull
$ gcb - git checkout -b
$ gm  - git merge
$ grb - git rebase
$ gpsup - git push --set-upstream origin $(current_branch)
$ gbda  - git branch --no-color --merged | command grep -vE "^(\+|\*|\s*(master|develop|dev)\s*$)" | command xargs -n 1 git branch -d
$ gfa - git fetch --all --prune
```
如果你忘记了这些或者其他你自己设置的别名，可以运行：
``` shell
$ alias
```
或者给出关键词进行搜索：
``` shell
$ alias grep <alias-name>
```

## 其他 Git 技巧
### 忽视文件 (Ignoring Files)
很多文件可能不需要存在于版本控制中，你可以设置[全局 gitignore 文件](http://egorsmirnov.me/2015/05/04/global-gitignore-file.html)
来忽视掉它们。需要忽视的文件可能是一些 `node_modules` 文件夹，
`.vscode` 或其他 IDE 的文件，以及一些 Python 的虚拟环境。

对于一些敏感信息，你可以使用环境变量文件存放，然后将它们添加到
项目根目录下的 `.gitignore` 文件中。

### 特殊文件 (Special Files)
你可能需要将一些文件标记为二进制文件，以便于 Git 可以将其忽视，
并且不用为它们产生冗长的差异性检测。Git 有一个 `.gitattributes` 文件
来实现这一操作。比如在一个 JavaScript 项目中，
你会在 `.gitattributes` 中添加一个 `yarn-lock.json` 或者 
`package-lock.json`，这样一来，在你每次更新时，
Git 不用每次都尝试记录它们的差异变化。
``` shell
# .gitattributes
package-lock.json binary
```

## Git 工作流
### Rebase vs. Merge
你的团队可能会从 rebase 和 merge 两种工作流中二选其一，
二者都有利弊，我曾见过这两种方式都可以产生很高的效率。
对于大多数情况，除非你 _真的_ 了解你正在做什么，
否则选择 merge 工作流就完事了。

当你主要使用 merge 来为产品更新迭代时，
仍然也可以高效的使用 rebase。
最常见的场景是，你正在一个 feature 上工作，
同时另外一个开发者 pull 了一个别的 feature 到 master。
你确实可以使用 `git merge` 将那些改变一起带上，
但是这样，你会对队友做的简单更改有一个额外的 commit。
你真正想做的是将你的提交 _重新提交_ 到最新的 master 的最上面。
``` shell
$ git rebase master
```
这条命令将给你一个更干净的 commit 历史。

深度解释它们之间的不同点可能需要一整篇论文来阐述，
因此，我建议你可以查阅 [the Atlassian docs](https://www.atlassian.com/git/tutorials/merging-vs-rebasing) 中有关这些差异的文章。

### 远程仓库设置 (Remote Repository Setting)
我对于 GitHub 和 GitLab 最为熟悉，但是其他远程仓库管理器也
应该支持这些设置。

#### 1. 在 merge 时删除分支
一旦有分支被 merge，你就不应该在关心这个分支，
因为它的历史将被映射到你的 master/dev 分支。
这一举措会显著的减少你所管理的分支数量。
也可以通过使用 `git fetch -all --prune` 更为高效的保持本地仓库的干净整洁。

#### 2. 防止直接 push 到 master
如果没有这个设置，很容易在 `git push` 时，忘记自己正在 master，
这会潜在的破坏你的产品，一点也不好。

#### 3. merge 前至少需要一次确认
取决于团队的大小，你可能需要在 merge 前需要多次的确认，
即使只是一个二人团队，最少也要确认一次。
你不用花费几个小时每行都看，但一般来说，
你的代码应该至少有两个人看过。
[反馈](https://dev.to/g_abud/effective-learning-and-feedback-loops-1110) 是学习和个人提升的关键。

#### 4. merge 前需要通过 CI 测试
(译者注：CI 即 Continuous Integration，[持续集成](https://gitbook.cn/gitchat/column/5aa795539c3cf94d49162f03/topic/5aaa1abe0bb9e857450e7a9e))

已损坏的改变不应该被 merge 到产品中。
测试人员无法 100% 的捕捉损坏的更改，因此需要自动执行这些检测。

### Pull 请求 (Pull Requests)
保持 Pull 请求小而简洁，理想情况下不超过几百行。
小而频繁的 Pull 请求会使得审阅过程更快，从而产生更多的无 bug 代码，
也会让你的队友更轻松，从而提升团队的效率，也更容易分享学习经验。
团队内部达成一个共同的承诺，承诺每天都花一些时间来审阅公开 Pull 请求。

我们都爱这样的审阅：
![reviewing]https://res.cloudinary.com/practicaldev/image/fetch/s--7vLB7w1a--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/gdcwrvn006gryk0xiox0.png)

如果你正在实现的特性在一段时间内都会处于损坏状态，
请使用[特性标签](https://www.martinfowler.com/articles/feature-toggles.html)来在产品中禁用它。
这将会防止你的特性分支和 dev/master 分支产生太大差异，
同时也允许你做更频繁，更小的 Pull 请求。不 merge 代码的时间越长，
以后 merge 的难度就越大。

最后，在你的 Pull 请求中放一个细节描述，如果必要的话，
可以放图片或者 GIF。如果你使用像 Jira 这样的工具管理票据
(tickets，译者注：没使过不太懂啥意思)，
描述中也可以包括 Pull 请求地址的票据的编号。
Pull 请求的描述和可视化做的越详细，可能你的队友就越想审阅你的代码，
而不是拖延着下次一定。

### 分支命名 (Branch Naming)
你的团队可能会提出分支命名的规范，以便于导航。
我喜欢每个分支以创建人的名字首字母开头，接着一个左斜杠，
然后是以短横线连接的分支描述。

这可能看起来微不足道，但是配合 tab 补全以及类似 grep 这样的工具一起使用，
这确实可以帮你找到并理解可能有的所有分支。

例如，我创建了一个新分支：
``` shell
$ git checkout -b gabud/implement-important-feature
```
一周后，当我忘记我之前给它起了什么名字，
我可以键入 `git checkout gabud`，然后按 tab 键，
我的 Z shell 就会向我展示所有我本地的分支以供选择，
而不用看我队友们的分支。

### 提交信息 (Commit Messages)
语言很重要，一般来说，我发现最好不要以破碎状态提交东西，
每一次提交都应该有一个简洁的信息，说明所做的更改。
按照官方 [Git 建议](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project)，我发现最好使用当前的命令式意义来提交信息。
可以将每个提交信息视为对 计算机/git 的命令，
以至于你可以将提交信息加到这句话后面：

**如果这个 commit 被应用，将会...**

在当前命令式意义上，良好的提交示例为：
``` shell
$ git commit -m "Add name field to checkout form"
```
现在可以这么读：“如果这个 commit 被应用，将会在表单中添加姓名域。”

### 最后的想法
这绝不是已经了解了 Git 的全部，建议查阅 [官方文档](https://git-scm.com/doc) 
和 `git help` 了解更多。
不要害怕向你的队友问一些 Git 的问题，
你会惊讶的发现大多数队友也有许多相同的问题。

那么你呢？哪个 Git 命令或者概念在你的工作流中最有用呢~
