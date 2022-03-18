---
title: "systemd 自动挂载samba"
date: 2022-03-18T11:30:51+08:00
tags: ['linux', 'samba', 'systemd']
categories: ['linux']
draft: false
---

## samba 挂载
目前有一个需求，系统重启时候需要自动挂载远程 samba 文件系统，起初使用 `fstab` 发现并不能成功挂载，考虑应该是和启动顺序有关系，当没有网络的时候是无法执行挂载操作的，或者说挂载会出错，后来发现 `systemd` 也提供自动挂载的功能，于是尝试了一下，发现是可以的。

## 具体步骤
1. 在 `/etc/samba/` 下创建 `credentials` 文件夹
```
mkdir /etc/samba/crendentials
```
2. 在`/etc/samba/crendentials`下创建比如名为`share0`的文件并加入如下配置
```
# vim /etc/samba/crendentials/share0
username=<samba登录用户名>
password=<samba对应的密码>
```
3. 创建systemd unit文件，内容如下
```
# vim /etc/systemd/system/mnt-share.mount
Description= mount a samba share
Requires=network-online.target
After=network-online.target systemd-resolved.service
Wants=network-online.target systemd-resolved.service

[Mount]
Name=mnt-share
What=//<远程samba-ip>/<远程samba路径>
Where=/mnt/share # <本机挂载路径>
Type=cifs
Options=x-systemd.automount,_netdev,credentials=/etc/samba/credentials/share,sec=ntlmssp,vers=1.0,rw,uid=1000,gid=1000,dir_mode=0777,file_mode=0777,iocharset=utf8
TimeoutSec=30

[Install]
WantedBy=multi-user.target
```
4. 测试是否可以挂载
```
systemctl start mnt-share.mount
```
如果有报错就需要尝试修改
5. 开机自动挂载
```
systemctl enable mnt-share.mount
```

> 注意：挂载路径(Where)必须和文件名(mnt-share0)对应，比如挂载到 `/mnt/share`，那么文件名必须是`mnt-share0.mount`

参考：[arch linux samba https://wiki.archlinux.org/index.php/samba#As_systemd_unit](https://wiki.archlinux.org/index.php/samba#As_systemd_unit)


