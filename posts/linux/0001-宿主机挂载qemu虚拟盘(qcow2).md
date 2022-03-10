---
title: "宿主机挂载qemu虚拟盘(qcow2)"
date: 2022-03-10T08:55:26+08:00
tags: ['linux', 'qemu']
categories: ['linux', 'qemu']
draft: false
---

## 在Linux宿主机上挂载 qemu 虚拟盘
### 下载 qemu-nbd 工具
```shell
    pacman -S qemu
或
    apt-get install qemu-utils
或
    yum install qemu-img
```

### 加载nbd模块并挂载
```shell
modprobe nbd max_part=8
qemu-nbd --connect=/dev/nbd0 <xxx.qcow2>
```

### 开始挂载nbd磁盘到文件系统
```shell
mount /dev/nbd0xx <挂载点>
```

### 卸载nbd
```shell
qemu-nbd --disconnect /dev/nbd0
```

## 将vmware的vmdk虚拟盘转为qemu支持的qcow2虚拟盘
### 合并vmware虚拟磁盘
如果vmware下虚拟机的磁盘是分块的，比如:`xxx-s001.vmdk`、`xxx-s002.vmdk`...

需要先合并为一个`vmdk`文件
```shell
vmware-vdiskmanager.exe -r <虚拟盘的路径> -t 0 <合并后盘的名字>
```

> vmware-vdiskmanager.exe 在vmware安装路径下

### vmdk转qcow2

在linux下执行(确保执行前安装有qemu环境和工具包)
```shell
qemu-img  convert  -f  vmdk  -O qcow2  <xxx.vmdk>  <xxx.qcow2>

```

