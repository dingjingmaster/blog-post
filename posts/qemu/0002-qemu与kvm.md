---
title: "Qemu与KVM"
date: 2023-04-13T15:31:50+08:00
tags: ['qemu', 'kvm', 'linux']
categories: ['qemu']
draft: false
---

## 虚拟化简介

### 虚拟化思想

虚拟化的主要思想是，通过分层将底层的复杂、难用的资源虚拟抽象成简单、易用的资源，提供给上层使用。

底层资源或者通过空间的分割、或者通过时间的分割，将下层资源通过一种简单易用的方式转换成另一种资源，提供给上层使用。

### 虚拟机简介

虚拟机就是提供一个完成用户指定任务的环境。

> 模拟器是另一种形式的虚拟机，典型的模拟器有 QEMU、Bochs。

模拟器实现两种方式：
1. 对源程序指令一条条分析、然后执行指令对应的操作
2. 通过二进制翻译实现，即先将程序中所有的源ISA指令翻译成目标ISA上具有同样功能的指令，然后在目标ISA上执行

### 系统虚拟化的历史

- 早期计算机不支持硬件层面的虚拟机，整个虚拟过程在应用层实现
- 后来CPU硬件层面开始支持虚拟化(x86在2005年开始支持)，2006年以色列Qumranet利用硬件虚拟化技术在Linux内核上开发了KVM。

目前，QEMU已经不仅仅作为一个模拟器了，而是作为以 QEMU-KVM 为基础的为云计算服务的系统虚拟化软件。

## 例子

### 程序
```asm
start:
mov     $0x48,%al
outb    %al,$0xf1
mov     $0x65,%al
outb    %al,$0xf1
mov     $0x6c,%al
outb    %al,$0xf1
mov     $0x6f,%al
outb    %al,$0xf1

mov     $0x0a,%al
outb    %al,$0xf1

hlt
```

### qemu

```c
/*************************************************************************
> FileName: qemu.c
> Author  : DingJing
> Mail    : dingjing@live.cn
> Created Time: Thu 13 Apr 2023 04:02:07 PM CST
 ************************************************************************/
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <sys/ioctl.h>
#include <linux/kvm.h>

int main (int argc, char* argv[])
{
    int ret = 0;
    struct kvm_sregs sregs;

    // 获取子文件系统的文件描述符
    int kvmFd = open ("/dev/kvm", O_RDWR);

    // 应用层检查版本是否支持
    ioctl(kvmFd, KVM_GET_API_VERSION, NULL);

    // 创建一个虚拟机，返回新建虚拟机的文件描述符
    int vmFd = ioctl(kvmFd, KVM_CREATE_VM, 0);

    // void* mmap(void* start, size_t length, int prot, int flags, int fd, off_t offset);
    unsigned char* ram = (unsigned char*) mmap (NULL, 0x1000, PROT_READ | PROT_WRITE | PROT_EXEC, MAP_SHARED | MAP_ANONYMOUS | MAP_EXECUTABLE, -1, 0);

    int kfd = open ("./test.bin", O_RDONLY);
    read (kfd, ram, 4096);

    struct kvm_userspace_memory_region mem = {
        .slot = 0,
        .flags = 0,
        .guest_phys_addr = 0,
        .memory_size = 0x1000,
        .userspace_addr = (unsigned long) ram,
    };

    // 为虚拟机指定内存
    ret = ioctl (vmFd, KVM_SET_USER_MEMORY_REGION, &mem);

    // 为虚拟机创建vcpu
    int vcpufd = ioctl (vmFd, KVM_CREATE_VCPU, 0);

    int mmap_size = ioctl(kvmFd, KVM_GET_VCPU_MMAP_SIZE, NULL);
    struct kvm_run* run = (struct kvm_run*) mmap (NULL, mmap_size, PROT_READ | PROT_WRITE, MAP_SHARED, vcpufd, 0);

    // 获取虚拟机 CPU 寄存器
    ret = ioctl (vcpufd, KVM_GET_SREGS, &sregs);
    sregs.cs.base = 0;
    sregs.cs.selector = 0;

    // 设置虚拟机 CPU 寄存器
    ret = ioctl(vcpufd, KVM_SET_SREGS, &sregs);
    struct kvm_regs regs = {
        .rip = 0,
    };

    ret = ioctl (vcpufd, KVM_SET_REGS, &regs);

    while (1) {
        ret = ioctl (vcpufd, KVM_RUN, NULL);
        if (-1 == ret) {
            printf ("exit unknow\n");
            return -1;
        }

        switch (run->exit_reason) {
        case KVM_EXIT_HLT: {
            puts("KVM_EXIT_HLT");
            return 0;
        }
        case KVM_EXIT_IO: {
            putchar(*(((char*) run) + run->io.data_offset));
            break;
        }
        case KVM_EXIT_FAIL_ENTRY:{
            puts("entry error");
            return -1;
        }
        default: {
            puts ("other error");
            printf ("exit reason: %d\n", run->exit_reason);
            return -1;
        }
        }
    }

    return 0;
}

```
