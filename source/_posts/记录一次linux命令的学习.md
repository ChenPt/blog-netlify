---
title: 记录一次linux命令的学习
category: linux
tags: [command, git]
date: 2022-07-12
---

记录一次linux命令的学习，主要涉及管道命令，grep，xargs，以及对git branch的-d, -D两者的比较。
<!-- more -->


# 记录一次linux命令的学习

## linux 命令实际应用

工作中，任务数量上去了之后，对应的git分支都很多，且很多feat分支的上游分支（远程分支）都已经在合并后被删除了，本地留存了太多无用分支，遂搜索查到一个命令用于批量删除本地分支。

`git branch | grep 'cpt-CSM-*' | xargs git branch -d`

这个复杂命令涉及到了linux的 `|`， `grep` ，`xargs`，以及我们熟悉的`git branch`

### 管道命令

管道命令使用 `|` 分隔开来，左侧命令产生一个标准输出，传递给右侧命令，且右侧命令要接受标准输入。管道命令只会处理前一个命令的正确输出，错误输出不处理。

### Grep命令

例子：`grep npm README.md` 

主要是在文件中查找文本，若没有指定文件名，则从标准输入中获取。

支持正则查询`grep -e 'cpt-CSM-*'`

### xargs

例子：`find *.md | xargs wc -l`

找到md结尾的文件，计算文件行数量。`wc` 命令不支持标准输入，所以使用`xargs` 将管道命令的上一个命令的`标准输入`转成`命令行参数。`

作用：当管道命令使用了无法接受标准输入（stdin）的命令时，可以通过xargs命令，将`标准输入`转化为`命令行参数` 。

个人猜测，xargs的原理是，将标准输入流的参数转化为参数列表（根据其options 参数，可能会涉及到参数分批，多子进程并发执行），将收集到的参数列表作为命令行参数，再用子命令程序执行命令。其他具体用法查阅[这篇文章](https://www.ruanyifeng.com/blog/2019/08/xargs-tutorial.html)

### git branch

`git branch` 可以打印当前本地仓库的所有分支。

`git branch -d [branchName]` 可以删除指定的分支。

`git branch -d` 和`git branch -D`的区别是，`-D`是强制性的。而`-d` 会检查当前删除分支是否已经有上游分支，或者当前HEAD指针是否有设置上游分支，”上游分支“是指托管在gitlab/github上的远程分支，具体信息查阅`git branch --help`输出。所以日常删除本地分支，还是使用`-d`参数，避免删除了还没提交远程的重要的分支。

## 结论

所以`git branch | grep 'cpt-CSM-*' | xargs git branch -d` 的实际意义就是，

1. `git branch` 输出本地仓库所有分支
2. `grep 'cpt-CSM-*'` 查找所有以`cpt-CSM-` 开头的分支
3. `xargs git branch -d` xargs转化标准输入为命令行参数，并将参数传递给`git branch -d` 命令并执行。