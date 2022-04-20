---
title: "rfkill —— 用户空间控制无线通讯命令"
date: 2022-04-20T11:19:06+08:00
tags: ['linux', 'rfkill', 'wlan', 'bluetooth']
categories: ['linux']
draft: false
---

## rfkill

rfkill 是一个小型的用户空间工具，用于查询 rfkill 开关、按钮和子系统接口的状态。

Linux 内核通过 `/dev/rfkill` 设备文件公开了 rfkill 子系统一些控制功能，允许用户空间模拟/监听/控制硬件 rfkill 设备。

## 具体使用

```shell
rfkill --help
```

## 参看
[https://wireless.wiki.kernel.org/en/users/Documentation/rfkill](https://wireless.wiki.kernel.org/en/users/Documentation/rfkill)
