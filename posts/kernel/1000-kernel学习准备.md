---
title: "Linux Kernel学习准备"
date: 2022-10-10T15:49:23+08:00
tags: ['linux', 'kernel']
categories: ['kernel']
draft: false 
---

## Linux 内核工程师

### 岗位分布
- 互联网
- 云计算
- 安全
- OS发行商
- 芯片原厂

### 技能点

- 内存管理、调度、实时性
- 进程、同步与并发
- 存储、文件系统
- 网络子系统
- 虚拟化、区块链、容器

## 驱动工程师

### 岗位分布

- 芯片原厂、方案厂商
- 嵌入式设备厂商

### 技能点

- 内核编译、移植
- 设备模型、驱动框架
- USB、PCI、I2C、UART
- 芯片手册、硬件电路
- 音视频编解码

### 驱动的衍化
- Misc驱动：一个驱动对应一个硬件、兼容性最差
- 字符驱动、设备驱动：读写方式
- 总线性驱动
    - 物理总线：USB、I2C、PCI
    - 虚拟总线：platform、device tree
- 驱动框架：framebuffer、input、DRM
- 用户态驱动：硬件抽象层HAL

## 驱动与内核的关系

### 驱动主要用来驱动各种硬件
- 驱动是内核代码的一部分：内核中80%的代码都是驱动
- 驱动不同的CPU、硬件平台、外设、热插拔设备

### 内核一般用来提供OS的各种服务
- Linux内核划分为：BSP、驱动、基础服务、应用服务
- 基础服务：进程、调度、内存管理、同步机制、中断
- 应用服务：虚拟化、容器、网络、存储、文件系统
- 随着应用场景的变化，新的功能会不断添加进来

## Linux内核版本变化

### Lnux-0.01
- 源文件: 88个文件、10239行代码
- 支持平台：X86

### Linux-5.4
- 源文件：48815个文件、超过2500万行代码（.h|.c|.S）
- 支持平台：X86、ARM、RISC-V、MIPS等20多个平台

## 知识体系

### Linux开发工具
- vim
- Makefile
- IDE
- debug

### 工具链
- 编译
- 链接
- 重定位
- 安装
- 运行

### 程序主战场
- 内存管理
- 堆
- 栈
- 堆栈溢出

### 处理器架构
- 指令集
- CPU 工作原理

## 基于Arch搭建实验平台(ARM A9 vexpress)
### 安装qemu-system

```shell
sudo pacman -S qemu-system-arm
```

### 编译内核镜像(5.10)

1. 下载linux-5.10镜像:[https://mirrors.tuna.tsinghua.edu.cn/kernel/v5.x](https://mirrors.tuna.tsinghua.edu.cn/kernel/v5.x)（注意拉到最下边找内核源码压缩包）
2. 编译
```shell
# 1. 创建文件夹
mkdir tftpboot
chmod 0777 tftpboot
tar xf linux-5.10.99tar.gz
cd linux-5.10.99
# vim Makefile +371 ; 将 ARCH 变量赋值为 arm，即：ARCH ?= arm
# vim Makefile +372 ; 将 CROCESS_COMPILE=arm-none-linux-gnueabihf-

make vexpress_defconfig

make zImage -j32
make modules -j32
make dtbs -j32
make LOADADDR=0x60003000 uImage -j32

cp arch/arm/boot/zImage ../tftpboot/
cp arch/arm/boot/uImage ../tftpboot/
cp arch/arm/boot/dts/vexpress-v2p-ca9.dtb ../tftpboot/
```
- `zImage` 为通用内核文件
- `modules` 内核模块
- `dtbs` 编译的设备树
- `uImage` 为专供 u-boot引导的内核
3. 启动脚本
`vim start.sh`
```shell
#!/bin/bash

qemu-system-arm                 \
    -M vexpress-a9              \
    -m 512M                     \
    -kernel zImage              \
    -dtb vexpress-v2p-ca9.dtb   \
    -nographic                  \
    -append "console=ttyAMA0"
```

### 制作根文件系统
> 使用 busybox 制作根文件系统

#### 编译、安装根文件系统

```shell
sudo mkdir -p arm-rootfs

# 下载 busybox 1.35.0 并解压
cd busybox-1.35.0

# 修改busybox的Makefile +191
# ARCH ?= arm
# CROCESS_COMPILE=arm-none-linux-gnueabihf-
# make menuconfig 设置安装路径，输入自己的 rootfs 根路径 `/data/learn-kernel/arm-rootfs`
make install
```

#### 完成完整的根文件系统构建
1. 安装动态链接库
```shell
mkdir arm-rootfs/lib
cp /usr/arm-none-linux-xxx/lib/*.so* arm-rootfs/lib -d
```

2. 创建设备节点
```shell
cd /data/learn-kernel/arm-rootfs/
mkdir dev
cd dev
sudo mknod -m 666 tty1 c 4 1
sudo mknod -m 666 tty2 c 4 2
sudo mknod -m 666 tty3 c 4 3
sudo mknod -m 666 tty4 c 4 4
sudo mknod -m 666 console c 5 1
sudo mknod -m 666 null c 1 3
```

3. 设置初始化进程/etc/rcS
```shell
cd /data/learn-kernel/arm-rootfs
mkdir -p etc/init.d
cd etc/init.d
touch rcS
chmod 777 rcS
```
`vim rcS`
```shell
#!/bin/sh

PATH=/bin:/sbin:/usr/bin:/usr/sbin
export LD_LIBRARY_PATH=/lib:/usr/lib

/bin/mount -n -t ramfs ramfs /var
/bin/mount -n -t ramfs ramfs /tmp
/bin/mount -n -t sysfs none  /sys
/bin/mount -n -t ramfs none  /dev

/bin/mkdir /var/tmp
/bin/mkdir /var/modules
/bin/mkdir /var/run
/bin/mkdir /var/log

/bin/mkdir -p /dev/pts
/bin/mkdir -p /dev/shm

/sbin/mdev -s
/bin/mount -a

echo "----------------------------------------------"
echo "******** welcome to vexpress board ***********"
echo "----------------------------------------------"
```

4. 设置文件系统`/etc/fstab`

```shell
cd /data/learn-kernel/arm-rootfs/etc
touch fstab
```
`vim fstab`
```shell
proc        /proc       proc        defaults        0   0
none        /dev/pts    devpts      mode=0622       0   0
mdev        /dev        ramfs       defaults        0   0
sysfs       /sys        sysfs       defaults        0   0
tmpfs       /dev/shm    tmpfs       defaults        0   0
tmpfs       /dev        tmpfs       defaults        0   0
tmpfs       /mnt        tmpfs       defaults        0   0
var         /dev        tmpfs       defaults        0   0
tamfs       /dev        ramfs       defaults        0   0
```
5. 设置初始化脚本 `/etc/inittab`

```shell
::sysinit:/etc/init.d/rcS
::askfirst:-/bin/sh
::ctrlaltdel:/bin/umount -a -r
```

6. 设置环境变量 `/etc/profile`

```shell
USER="root"
LOGNAME=$USER

export HOSTNAME=`cat /etc/sysconfig/HOSTNAME`
export USER=root
export HOME=/root
export PS1="[$USER@$HOSTNAME\w]\#"

PATH=/bin:/usr/bin:/sbin:/usr/sbin
LD_LIBRARY_PATH=/lib:/usr/lib:$LD_LIBRARY_PATH
export PATHLD_LIBRARY_PATH
```

7. 增加主机名 `/etc/sysconfig/HOSTNAME`
```shell
vexpress
```

8. 创建剩余文件夹
```shell
mkdir mnt proc root sys tmp var
```

#### 封装构建好的根文件系统并挂载
```shellc
mkdir temp
sudo dd if=/dev/zero of=rootfs.ext3 bs=1M count=32
sudo mkfs.ext3 rootfs.ext3
sudo mount -t ext3 rootfs.ext3 temp/ -o temp
sudo cp -r /data/learn-kernel/arm-rootfs/* temp/
sudo umount temp
sudo mv rootfs.ext3 tftpboot
```

修改 `start.sh`
```shell
#!/bin/bash

qemu-system-arm                 \
    -M vexpress-a9              \
    -m 512M                     \
    -kernel zImage              \
    -dtb vexpress-v2p-ca9.dtb   \
    -nographic                  \
    -append "root=/dev/mmcblk0 rw console=ttyAMA0"      \
    -sd rootfs.ext3
```
**至此，环境搭建完成**

### 使用u-boot加载内核
> 完成上述步骤后之所以能直接引导内核是因为 qemu 自带bootloader功能。
#### u-boot编译和安装

```shell
mkdir u-boot
# 下载并解压u-boot
cd u-boot-2022.07
vim Makefile +272
# 修改: ARCH = arm
# 修改: CROSS_COMPILE=arm-none-linux-gnueabihf-
make vexpress_ca9x4_defconfig
make -j32
```
将编译好的`u-boot` 放到`tftpboot`目录下(与start.sh同级)修改 `start.sh`
```shell
# 使用 u-boot
qemu-system-arm                 \
    -M vexpress-a9              \
    -kernel u-boot              \
    -nographic                  \
    -m 512M
```
可以看到 `qemu` 可以加载 `u-boot`

<br/>
接下来使 u-boot 启动后，让 u-boot 通过 tftp 引导内核。

`qemu` --加载--> `u-boot` --加载--> `kernel+根文件系统`

为此需要实体机中安装:
1. tftp环境
2. 构建网桥，让qemu虚拟机内部可以访问

#### 安装tftp
```shell
pacman -S tftp-hpa bridge-utils
# 设置 tftp 路径 
vim /etc/default/tftpd-hpa
# 修改内容如下：
# TFTP_USERNAME="tftp"
# TFTP_DIRECTORY="/data/learn-kernel/fs"
# TFTP_ADDRESS="0.0.0.0"
# TFTP_OPTIONS="-l -c -s"
```
文件复制
```shell
cd /data/learn-kernel/tftpboot/fs
cp /<path>/uImage .
```
重启tftpd
```shell
systemctl restart tftpd.service
```
#### 修改网卡信息，设置桥接
```shell
ifconfig # 查看网卡名 我的是:`wlo1`
sudo vim /etc/netplan/0
```
## 未完待续... 
> 需要补课 Linux 网络部分...
