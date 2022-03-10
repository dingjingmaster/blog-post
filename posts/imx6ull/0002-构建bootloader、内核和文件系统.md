---
title: "构建bootloader、内核和文件系统"
date: 2022-03-09T13:39:38+08:00
tags: ['linux', 'imx6ull']
categories: ['linux']
draft: false
---

## 准备环境
1. 工作机 `linux` 上要准备好 `arm-gcc` 编译工具链(如果没有则看[imx6ull概念和环境搭建](/2022/03/0001-基础概念与环境搭建/))
2. 工作机 `linux` 上可以通过串口连接到 `arm 开发板`(如果没有则看[imx6ull概念和环境搭建](/2022/03/0001-基础概念与环境搭建/))
3. 工作机 `linux` 上准备分别准备好 `buildroot` `u-boot` `kernel` 的源码

## 编译`u-boot`
### `u-boot`介绍
`u-boot`属于 bootloader 的一种实现。bootloader 是在操作系统运行之前运行的一段代码，用于引导操作系统。
### `u-boot`编译
这次编译使用 imx6ull 资料里提供的 `Uboot-2017.03` 源码
```shell
make distclean


```


## 编译`kernel`

1. 找到 `imx-linux4.9.88` 并执行命令
  > 这里需要注意，imx6ull pro 处理器是 arm v7l 是 32 位系统
  ```shell
  export ARCH=arm
  export CROSS_COMPILE=arm-none-linux-gnueabihf-
  make mrproper
  make 100ask_imx6ull_defconfig
  make zImage -j8
  make dtbs
  ```
## 编译`buildroot`

