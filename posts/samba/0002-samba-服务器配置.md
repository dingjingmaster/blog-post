---
title: "Samba 服务器配置"
date: 2022-05-13T10:53:54+08:00
tags: ['samba', 'linux']
categories: ['samba']
draft: false
---

## 配置文件

| 配置文件                                         | 说明                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| /etc/samba/smb.conf                              | samba的主要配置文件，可设置全局参数和共享目录的参数          |
| /etc/samba/lmhosts                               | 通过hostname来访问samba                                      |
| /etc/samba/smbusers                              | 由于windows和linux里的管理员和访客账号名称不一致，可使用此配置文件来设置一个映射，比如administrator映射成root |
| /etc/sysconfig/samba                             | 配置smbd，nmbd启动时带的参数                                 |
| /var/lib/samba/private/{passdb.tdb, secrets.tdb} | 管理samba的用户账号/密码时，会用到的数据库档案               |

## 可用命令

| 命令               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| smbd               | smbd提供文件和打印共享服务器                                 |
| nmbd               | nmbd提供NetBIOS名称服务和浏览支持，帮助客户端定位服务器，处理所有基于UDP的协议 |
| tdbdump,tdbtool    | samba使用了tdb数据库，可以使用tdb工具来查看数据库内容        |
| smbstatus          | 查看samba的状态                                              |
| smbpasswd, pdbedit | 服务器功能，用于管理samba的用户账号和密码，早期是使用smbpasswd命令，后来因为使用了tdb数据库，所以推荐使用pdbedit命令来管理用户数据 |
| mount.cifs         | 用来挂载分享目录                                             |
| smbclient          | samba客户端命令                                              |
| nmblookup          | 查找NetBIOS name                                             |
| smbtree            | 未知，可能是用来查找网络邻居的吧                             |
| testparm           | 验证smb.conf文件的内容是否合法                               |

## 工作模式

samba 共有5种工作模式，分别为：

1. share，用户对samba服务器的访问不需要身份验证，允许匿名访问，用户的访问权限仅由相应用户对共享文件的访问权限决定
2. user，使用用户名和密码访问samba服务器
3. server，使用另外一台服务器专门用来做身份验证，samba服务只提供文件和打印机共享服务
4. domain，域模式<待补充>
5. ads，<待补充>

通过设置security选项即可设置samba的工作模式：security = share

## 配置项

### 全局配置

```
[global]
   workgroup = WORKGROUP
   dns proxy = no
   log file = /var/log/samba/%m.log
   max log size = 1000
   client min protocol = SMB2
   server role = standalone server
   passdb backend = tdbsam
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *New*UNIX*password* %n\n *ReType*new*UNIX*password* %n\n *passwd:*all*authentication*tokens*updated*successfully*
   pam password change = yes
   map to guest = Bad Password
   usershare allow guests = yes
   name resolve order = lmhosts bcast host wins
   security = user
   guest account = nobody
   usershare path = /var/lib/samba/usershare
   usershare max shares = 100
   usershare owner only = yes
   force create mode = 0070
   force directory mode = 0070
```

### 共享目录配置

#### 不需要密码的共享

需要将全局参数中的security设置成share(暂不清楚，在user工作模式下通过设置guest ok好像也可以，需要验证) 最小化配置：

```
[test]
	comment = test
	path = /tmp
	read only = no
	guest ok = yes
	create mask = 644
```

其中： read only默认为yes，表示只允许读，不允许写，所以需要修改 guest ok默认是no，表示不允许匿名访问 create mask默认是744，导致客户端创建的文件都是可执行文件，所以需要修改

> 注意： writable和writeable是同义词 writeable和read only是反义同义词 writeable默认为no read only默认为yes 完整配置需要配置available和browseable，不过这两个默认都是yes

#### 用户名/密码方式的共享

需要将全局参数中的security设置成user

```
[win]
	comment = win
	path = /home/win
	read only = yes
	create mask = 644
	valid users = win
```

这种方式首先需要使用root权限添加一个账户，然后使用smbpasswd -a xxx在samba数据库添加此用户的samba密码  输入smbpasswd -a xxx 时会直接让用户设置这个账户的samba密码 这个用户信息保存在tdb数据库里  修改密码：root权限下输入smbpasswd user_name即可修改user_name的samba密码

## 配置文件验证

使用testparm可以验证smb.conf文件的内容是否合法
