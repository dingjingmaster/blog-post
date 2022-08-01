---
title: "0010 Linux挂载qcow2"
date: 2022-07-24T22:02:25+08:00
tags: ['linux', 'qcow2']
categories: ['linux']
draft: false
---

1. 确定linux是否有`nbd`模块
```shell
modinfo nbd
```

2. 插入`nbd` 模块
```shell
sudo modprobe nbd
```
3. 使用`qemu-nbd` 工具关联 qcow2 文件和 nbd 设备节点
```shell
sudo qemu-nbd  -c  /dev/nbd0  <qcow2的路径>
```

4. 通过`/dev/nbd0px`按需挂载 qcow2 设备


