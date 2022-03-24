---
title: "Wireshark 使用初步"
date: 2022-03-23T15:06:39+08:00
tags: ['linux', 'tool']
categories: ['linux']
draft: false
---

## wireshark 是什么

Wireshark(前称Ethereal)是一个网络封包分析软件。网络封包软件的功能主要是截取网络封包，并尽可能显示出最为详细的网络封包资料。Wireshark使用 WinPCAP 作为接口，直接与网卡进行数据报文交换。

## wireshark 使用
首先进入 wireshark，启动时候需要使用命令行以 `root` 启动
> 非root用户无法抓取网卡上所有的包

### 1. 选择网卡
选择菜单栏上 Capture -> Option，选中使用的网卡，点击 start 开始抓包
<div align=center><img src='/pic/tools/wireshark-1.png'/></div>
<center>选择网卡</center>

### 2. 添加过滤条件
比如仅需要查看 `www.baidu.com` 的数据包，通过 `ping` 命令获取对应的 `ip`，在 wireshark 中设置 `ip` 过滤条件就可以看到。
> `ip.addr == 119.75.217.26 and icmp` 表示只显示ICPM协议且源主机IP或者目的主机IP为119.75.217.26的数据包
<div align=center><img src='/pic/tools/wireshark-2.png'/></div>
<center>添加过滤条件</center>

## wireshark 界面说明

<div align=center><img src='/pic/tools/wireshark-3.png'/></div>
<center>wireshark界面说明</center>

### 数据详细区说明

- Frame: 物理层的数据帧概况
- Ethernet II: 数据链路层以太网帧头部信息
- Internet Protocol Version 4: 互联网层IP包头部信息
- Transmission Control Protocol:  传输层T的数据段头部信息，此处是TCP
- 之后的都属于应用层
<div align=center><img src='/pic/tools/wireshark-4.png'/></div>
<center>wireshark界面说明</center>

## wireshark 设置过滤条件

> 需要注意的是，wireshark 既可以设置`抓包过滤条件`，也可以设置`显示过滤条件`，前者限制大，用于减少捕获数据包的数量；后者用于在报文中隐藏某些报文。
> 抓包过滤器只能在抓包前设置，无法动态修改，显示过滤器则没有此限制

1. 设置`抓包过滤条件`，(如果已经开始抓包，需要先停止...)菜单栏 `Capture` -> `Options` -> `Input` -> `Capture filter for selected interfaces` 设置配置条件，同时选中要抓取的网卡，点击此页面右下角的 `Start` 即可开始抓包
2. 设置`显示过滤条件`，在菜单栏下边有个文本输入框(`Apply a display filter ... <Ctrl-/>`) 在这里配置显示过滤条件

### 抓取过滤类型(抓包过滤)
|类型|说明|
|---|---|
|host|让wireshark只抓取源于或发往由标识符host所指定的主机名或IP地址的流量|
|net|让wireshark只抓取源于或发往由标识符net所标识的IPv4/IPv6网络号的流量|
|port|让wireshark只抓取源于或发往由标识符port指定端口号的流量包|
|portrange|让wireshark只抓取源于或发往由标识符portrange指定端口范围的流量包|

### 过滤方向(显示/抓包过滤)
|方向|说明|
|---|---|
|src|发送端条件|
|dst|目标端条件|

### 过滤协议(显示过滤)
|协议|说明|
|---|---|
|ether||
|fddi||
|tr||
|wlan||
|ip||
|ip6||
|arp||
|rarp||
|decnet||
|sctp||
|tcp||
|udp||
|http||
|...||

### 逻辑运算符(显示过滤)
|运算符|说明|
|---|---|
|`&&` 或 `and`|两者等价`与`逻辑|
|`\|\|` 或 `or`|两者等价`或`逻辑|
|`!` 或 `not`|两者等价`非`逻辑|

### 比较操作符(显示过滤)
|运算符|说明|
|---|---|
|`==`|等于|
|`!=`|不等于|
|`>`|大于|
|`<`|小于|
|`>=`|大于等于|
|`<=`|小于等于|

## 例子
### 抓包过滤条件
1. 抓取百度的包
```
host www.baidu.com
```
或者使用 `ping  www.baidu.com` 得到百度网站的ip (`220.181.38.150`)再使用命令
```
net 220.181.38.150
```

2. 抓取指定ip范围内的包
```
net 192.168.0.0/24
```
或
```
net 192.168.0.0 mask 255.255.255.0
```

3. 抓取从指定 IP 范围发出的包
```
src net 192.168.0.0/24
```
或
```
src net 192.168.0.0 mask 255.255.255.0
```

4. 抓取发往目标 IP 范围的包
```
dst net 192.168.0.0/24
```
或
```
dst net 192.168.0.0 mask 255.255.255.0
```

5. 抓取指定端口`53`的包(DNS服务)
```
port 53
```

6. 抓取指定服务器上非 HTTP 和非 SMTP 的包
```
host www.example.com and not (port 80 or port 25)
host www.example.com and not port 80 and not port 25
```

7. 抓取除了 ARP 和 DNS 的包
```
port not 53 and not arp
```

8. 抓取指定端口范围的包
```
(tcp[0:2] > 1500 and tcp[0:2] < 1550) or (tcp[2:2] > 1500 and tcp[2:2] < 1550)
```
或
```
tcp portrange 1501-1549
```

9. 值抓取EAPOL类型以太网的包
```
ether proto 0x888e
```

10. 拒绝向链路层发现协议发送的以太网帧
```
not ether dst 01:80:c2:00:00:0e
```

11. 仅捕获IPv4流量-最短的过滤器，但有时非常有用，以摆脱较低的层协议，如ARP和STP
```
ip
```

12. 仅捕获单播流量-如果你只想看到与你的机器之间的流量，而不捕获广播和多播
```
not broadcast and not multicast
```

13. 捕获IPv6“所有节点”(路由器和邻居通告)流量。可以用来找到流氓RAs
```
dst host ff02::1
```

14. 捕获HTTP GET请求。这将查找TCP头后面的字节'G'、'E'、'T'和' '(十六进制值47、45、54和20)。"tcp[12:1] & 0xf0) >> 2"计算出tcp报头长度
```
port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420
```

### 显示过滤器
1. 仅显示 SMTP(25端口)和 ICMP 数据包
```
 tcp.port eq 25 or icmp
```
2. 只显示局域网(192.168.x.x)的流量，工作站和服务器之间-没有Internet:
```
ip.src==192.168.0.0/16 and ip.dst==192.168.0.0/16
```

3. TCP buffer full - Source指示Destination停止发送数据
```
tcp.window_size == 0 && tcp.flags.reset != 1
```
<br/>
<br/>

[参考：https://wiki.wireshark.org/DisplayFilters](https://wiki.wireshark.org/DisplayFilters)

[参考: https://wiki.wireshark.org/Home](https://wiki.wireshark.org/Home)
