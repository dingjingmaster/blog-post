---
title: "git rebase与merge的区别"
date: 2022-05-31T10:32:59+08:00
tags: ['git', 'git merge', 'git rebase']
categories: ['git']
draft: false
---

## 简单概括

`git rebase` 和 `git merge` 这两个命令旨在将更改代码从一个分支合并到另一个分支，只是两者合并方式不一样。

- 融合代码到公共分支的时候用 `git merge`，而不能使用 `git rebase`
- 融合代码到个人分支的时候用 `git rebase`， 可以不污染分支的提交记录，形成简洁的线性提交历史记录

> `git rebase aa bb`, 会把 `bb` 合并到 `aa` 的头部
> `git checkout main && git rebase bb` 会把 `main` 放到 `bb` 头部

### rebase 详细过程
<div align=center><img src='/pic/git/17.png'/></div>
<center>初始状态</center>

<div align=center><img src='/pic/git/18.png'/></div>
<center>git checkout -b aa && git commit -m "aa"</center>

<div align=center><img src='/pic/git/19.png'/></div>
<center>git checkout -b main && git commit -m "main"</center>

<div align=center><img src='/pic/git/20.png'/></div>
<center>git rebase aa main</center>

<div align=center><img src='/pic/git/21.png'/></div>
<center>git commit -m "rebase"</center>

> 上图中 `main` 原来的 HEAD 是 `c3`，rebase 之后变成了 `c3'`，`main` 分支到了 `c2` 之后，`c3'` 与 `c3` 相比就是 commit id 发生了变化

## 场景模拟

假设当前我们有master和feature分支，当你在专用分支上开发新 feature 时，然后另一个团队成员在 master 分支提交了新的 commits，这种属于正常的Git工作场景。如下图：

<div align=center><img src='/pic/git/1.png'/></div>
<center>实际工作场景</center>

现在，假设在 master 分支上的新提交与你正在开发的 feature 相关，需要master分支将新提交的记录合并到你的 feature 分支中。

### git merge 合并master到feature
```
git checkout feature
git merge master
```
或者缩写一句
```
git merge feature master
```
<div align=center><img src='/pic/git/2.png'/></div>
<center>git merge feature master</center>

由此可见，`git merge` 会在 `feature` 分支中新增一个新的 `merge commit`，然后将两个分支的历史联系在一起
- 使用 `merge` 是很好的方式，因为它是一种非破坏性的操作，对现有分支不会以任何方式被更改
- 另一方面，这也意味着 `feature` 分支每次需要合并上游更改时候，都会产生一个额外的合并提交。
- 如果 `master` 提交非常活跃，这可能会严重污染你的 `feature` 分支历史记录。不过这个问题可以使用高级选项 `git log` 来缓解

### git rebase 合并master到feature
```
git checkout  feature
git rebase master
```
缩写为一句就是：
```
git rebase feature master
```
<div align=center><img src='/pic/git/3.png'/></div>
<center>git rebase feature master</center>

- `rebase` 会将整个 `feature` 分支移动到 `master` 分支的顶端，从而有效地整个了所有 `master` 分支上的提交。
- 但是，与`merge` 提交方式不同，`rebase`通过为原始分之中的每个提交创建全新的 commits 来重写项目历史记录，特点是仍然会在 `feature` 分支上形成线性提交。
- `rebase` 的主要好处是可以获得更清晰的项目历史。首先，它消除了 `git merge` 所需的不必要的合并提交；其次，正如你在上图中所看到的，`rebase` 会产生完美线性的历史记录，你可以在 `feature` 分支上没有任何分叉的情况下一直追寻到项目的初始提交。

## git rebase 原理

将 `master` 分之代码合并到 `feature` 上：

<div align=center><img src='/pic/git/4.png'/></div>

这边需要强调一个概念：`reapply(重放)`，使用 `rebase` 并不是简单的好像用 `ctrl-x/ctrl-v` 一样进行剪切复制一样，`rebase`会依次将你所要操作的分支的所有提交应用到目标分之上。

合并过程如下图：

<div align=center><img src='/pic/git/5.png'/></div>
<center>git rebase 合并图</center>

从上图可以看出，在对特征分之进行 `rebase` 之后，其等效于创建了新的提交。并且老的提交也没有被销毁，只是简单的不能再被访问或者使用。

> 实际上在执行 `rebase` 的时候，有两个隐含的注意点：
> 1. 在重放之前的提交的时候，Git会创建新的提交，也就是说即使你重放的提交与之前的一模一样Git也会将之当做新的独立的提交进行处理。
> 2. git rebase并不会删除老的提交，也就是说你在对某个分支执行了rebase操作之后，老的提交仍然会存放在.git文件夹的objects目录下。

## 如何选择 `git merge` 和 `git rebase`

根据上面的对比可知：
- `git merge` 分支代码合并后不破坏原分支的代码提交记录，缺点就是会产生额外的提交记录并进行两条分支的合并
- `git rebase` 优点是无须新增提交记录到目标分支，rebase后可以将对象分支的提交历史续上目标分支上，形成线性提交历史记录，进行review的时候更加直观
- `git merge` 如果有多人进行开发并进行分支合并，会形成复杂的合并分支图

> 问题：既然rebase如此有用，那么可以使用rebase完全取代merge吗？ 答案：不能！
> `git rebase` 黄金法则：不能在一个共享的分支上进行 `git rebase` 操作

## rebase 的黄金法则
> 为什么不能在一个共享的分支上进行`git rebase`操作呢？

所谓共享的分支，即是指那些存在于远端并且允许团队中的其他人进行Pull操作的分支，比如我们Git工作的master分支就是最常见的公共分支。

假设现在Bob和Anna在同一个项目组中工作，项目所属的仓库和分支大概是下图这样：
<div align=center><img src='/pic/git/6.png'/></div>

现在Bob为了图一时方便打破了原则(使用了`git rebase`)，正巧这时Anna在特征分支上进行了新的提交，此时的结构图大概是这样的：
<div align=center><img src='/pic/git/7.png'/></div>

当Bob推送自己的分支到远端的时候，现在的分支情况如下：
<div align=center><img src='/pic/git/8.png'/></div>

然后呢，当Anna也进行推送的时候，她会得到如下的提醒，Git提醒Anna她本地的版本与远程分支并不一致，需要向远端服务器拉取代码进行同步：
<div align=center><img src='/pic/git/9.png'/></div>

在Anna提交之前，分支中的Commit序列是如下这样的：
```
A--B--C--D'   origin/feature // GitHub

A--B--D--E    feature        // Anna
```

在进行Pull操作之后，Git会进行自动地合并操作，结果大概是这样的：
<div align=center><img src='/pic/git/10.png'/></div>

这个第M个提交即代表着合并的提交，也就是Anna本地的分支与Github上的特征分支最终合并的点，现在Anna解决了所有的合并冲突并且可以Push她的代码，在Bob进行Pull之后，每个人的Git Commit结构为：
<div align=center><img src='/pic/git/11.png'/></div>

看到上面这个混乱的流线图，相信你对于Rebase和所谓的黄金准则也有了更形象深入的理解。

假设下还有一哥们Emma，第三个开发人员，在他进行了本地Commit并且Push到远端之后，仓库变为了：
<div align=center><img src='/pic/git/12.png'/></div>

另外，相信你也注意到，在远端的仓库中存有大量的重复的Commit信息，这会大大浪费我们的存储空间。

因此，**不能在一个共享的分支上进行Git rebase操作,避免出现项目分支代码提交记录错乱和浪费存储空间的现象。**

## 使用 rebase 合并多次提交记录

> rebase和merge不同的作用还有一个就是合并分支多次提交记录。

在分支开发的过程中，我们常常会出现为了调试程序而多次提交代码记录，但是这些记录的共同目的都是为了解决某一个需求，所以，是否可以将这些记录合并起来为一个新的记录会更方便进行代码的review呢？

### 合并记录
1. 尝试合并分支的最近 4 次提交纪录
```
git rebase -i HEAD~4
```

2. 这时候，会自动进入 vi 编辑模式：
<div align=center><img src='/pic/git/13.png'/></div>
进入编辑模式，第一列为操作指令，第二列为commit号，第三列为commit信息。
- pick：保留该commit
- reword：保留该commit但是修改commit信息
- edit：保留该commit但是要修改commit内容
- squash：将该commit和前一个commit合并
- fixup：将该commit和前一个commit合并，并不保留该commit的commit信息
- exec：执行shell命令
- drop：删除该commit

按照如上命令来修改你的提交记录：
```
p 799770a add article
s 72530e4 add article
s 53284b1 add article
s 9f6e388 add article
```
成功合并了四条记录为一条：
<div align=center><img src='/pic/git/14.png'/></div>

如果保存的时候，你碰到了这个错误：
```
error: cannot 'squash' without a previous commit
```
说明你在合并记录的时候顺序错误了，压缩顺序应该是从下往上，而不是从上往下，否则就会触发上面的错误。也就是以新记录为准。

中途出现异常退出了 vi窗口，执行下面命令可以返回编辑窗口：
```
git rebase --edit-todo
```
继续编辑合并记录的操作，修改完保存一下：
```
git rebase --continue
```

## git pull -rebase 的应用

### 场景
同事都基于git flow工作流的话都会从develop拉出分支进行并行开发，这里的分支可能是多到数十个，然后彼此在进行自己的逻辑编写，时间可能需要几天或者几周。

在这期间你可能需要时不时的需要pull下远程develop分支上的同事的提交。这是个好的习惯，这样下去就可以避免你在一个无用的代码上进行长期的开发，回头来看这些代码不是新的代码。甚至是会面临很多冲突需要解决，而这个时候你可能还需要对冲突的部分代码进行测试和问题解决，你在有些时候pull代码的时候会有这样的一个提示：
<div align=center><img src='/pic/git/15.png'/></div>

通常习惯性的你可能会，”esc ：wq“，直接默认commit注释。然后你的commit log就多了一笔很不好看的log。
<div align=center><img src='/pic/git/16.png'/></div>

### 如何移除多余的 merge commit

> 很简单，只要你在pull时候需要习惯性的加上—rebase参数，这样可以避免很多问题。

`git pull --rebase`可以免去分支中出现大量的merge commit，基本原理就像上面rebase一样，合并过程会直接融合到目标分支的顶端，形成线性历史提交。

> 注意： 在公共的分支上，例如`master`仍然要遵守rebase黄金原则，不用使用git pull --rabase进行代码的拉取，更改代码的历史提交记录。





## 最后
很好的git学习网站：[git 命令学习](https://learngitbranching.js.org/?locale=zh_CN)
