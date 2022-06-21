---
title: "Golang无法识别GOROOT"
date: 2022-06-21T09:15:28+08:00
tags: ['go', 'goland', 'GOROOT']
categories: ['IDE']
draft: false
---

## 问题
已经安装好go并配置好GOROOT和GOPATH之后，在Goland无法自动识别出GOROOT。在Goland设置选择go安装目录下，提示"Go SDK 非法的家目录"

## 解决
修改`${GROOT}/src/runtime/internal/sys/zversion.go`，添加go SDK 对应的版本号信息: `const TheVersion = 'go1.18.3'`
