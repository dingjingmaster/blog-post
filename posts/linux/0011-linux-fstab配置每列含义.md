---
title: "Linux Fstab配置每列含义"
date: 2022-08-01T17:17:36+08:00
tags: ['linux', 'fstab']
categories: ['linux']
draft: false
---

|列数|含义|
|---|---|
|1|设备名|
|2|挂载点|
|3|文件系统格式(比如:`ext`, `ext2`, `msdos`, `iso9660`, `nfs`, `swap`), 字段来源:`/proc/filesytems`|
|4|挂载参数，详细看 `man mount`，不同文件系统挂载参数可能不同。<br/>常见参数:<br/>`auto`: 系统自动挂载;<br/>`defaults`: rw,suid,dev,exec,auto,nouser<br/>`noauto`:开机不自动挂载;<br/>`nouser`: 只有超级用户可以挂载;<br/>`ro`: 只读挂载; <br/>`rw`: 读写挂载; <br/>`user`: 任何用户都可挂载; <br/>`注意: 光驱和软驱只有再装有介质时候才能挂载，因此它是noauto`|
|5|转储参数(0: 不备份; 1: 要备份，一般根分区都要备份)|
|6|自检顺序(0: 不自检; 1和2都是要自检，根分区一定设为1,其它分区要自检只能是2)，数字越小越先检查，数字相同则同时检查|

