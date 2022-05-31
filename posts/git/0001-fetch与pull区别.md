---
title: "git fetch与pull区别"
date: 2022-05-31T09:40:03+08:00
tags: ['git', 'fetch', 'pull']
categories: ['git']
draft: false
---

## 简单概括

- `git fetch` 是将远程主机的最新内容拉到本地，用户在检查了以后决定是否合并到工作本机分支中。
- `git pull` 则是将远程主机的最新内容拉下来后直接合并，即：`git pull` <==> `git fetch` + `git merge`，这样可能会产生冲突，需要手动解决。

## git fetch 用法
```
# 将某个远程主机的更新全部取回本地
git fetch  <远程主机名>                 

# 只去会特定分之的更新，可以增加分支名
git fetch  <远程主机名>  <分支名>

# 取回更新后，会返回一个 `FETCH_HEAD`，指向的是某个 branch 在服务器上的最新状态，我们可以在本地通过它查看刚取回的更新信息
git log -p FETCH_HEAD
```

## git pull 用法
```
git pull  <远程主机名>  <远程分支名>:<本地分支名>

# 如果远程分支是与当前分支合并，则冒号后面的部分可以省略
git pull  origin xxx
```
