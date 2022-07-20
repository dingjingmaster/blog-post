---
title: "Linux终端git无法联网"
date: 2022-07-19T22:38:41+08:00
tags: ['git']
categories: ['git']
draft: false
---

## 问题描述

电脑使用了`clash`代理工具，配好环境之后浏览器可以访问`谷歌`和`github`，但是终端无法推送代码到github

## 解决方法
打开终端执行如下命令：
```
git config --global http.proxy 'sockets5://127.0.0.1:7891'
git config --global https.proxy 'sockets5://127.0.0.1:7891'
```

