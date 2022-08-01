---
title: "Vmware转qemu"
date: 2022-07-24T22:44:23+08:00
tags: ['linux', 'qemu', 'vmware']
categories: ['linux']
draft: false
---

从 windows 换成 linux 之后发现 vmware 在linux 下运行很卡，于是考虑使用 qemu，图形界面开源安装 `virt-manager` 方便使用，而 vmware 的虚拟磁盘用的是 `vmdk`， 需要转为qemu支持的 `qcow2`。

<br/>

使用 `qemu-img` 完成 `vmdk` 转 `qcow2`

```shell
qemu-img convert -f vmdk <此处写vmware虚拟磁盘 xxx.vmdk 文件路径> -O qcow2 <此处写转换后要保存的 qemu 虚拟文件路径>
```

亲测原先vmware装的linux虚拟机可以很容易转成功，但是vmware虚拟机是windows10转换后没办法引导，以后解决引导问题再补充。



