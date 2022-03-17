---
title: "Buildroot"
date: 2022-03-15T20:35:39+08:00
tags: ['linux', 'imx6ull', 'buildroot']
categories: ['linux']
draft: false
---

## 什么是buildroot
buildroot 是一个简单、高效、易用的工具，能够利用交叉编译生成嵌入式linux系统

## buildroot 项目

### 拉取buildroot源码
```
https://github.com/imx6ull-pro/buildroot.git
```

#### buildroot 配置编译选项
##### 查询可用目标系统
```
make list-defconfigs
```

##### 选定目标系统
```
make imx6ullevk_defconfig
```

##### 打开配置菜单
```
make menuconfig
# 或
make nconfig
```

##### 查看帮助
```
make help
```

