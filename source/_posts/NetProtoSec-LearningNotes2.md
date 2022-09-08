---
title: 主机和端口扫描
date: 2022-09-07 19:51:56
tags: 
  - 网络安全
  - Wireshark
  - Nmap
categories:
  - 网络协议安全学习笔记
description: 国科大《网络协议安全》实验1 —— 主机和端口扫描
cover: https://s3.bmp.ovh/imgs/2022/09/03/532c30c044da0f35.jpg
---

> 实验环境：Kali+Wireshark+Nmap
> - Kali：是一个基于Debian的Linux发行版，预装了许多渗透测试软件，是一个很酷的“黑客操作系统”；
> - Wireshark：是一个网络封包分析软件，功能是截取网络封包，并尽可能显示出最为详细的网络封包资料。Wireshark使用WinPCAP作为接口，直接与网卡进行数据报文交换；
> - Nmap：即Network Mapper，是Linux下的网络扫描和嗅探工具包。

# 实验内容
1.在Nmap中，使用nmap –sP xx指令实现主机扫描，用wireshark抓取扫描过程中的ARP请求和响应报文；通过ping指令发起ICMP扫描，用wireshark抓取扫描过程中的ICMP报文；

2.在Nmap中，使用nmap –sS xx指令实现端口扫描，用wireshark抓取扫描过程中的TCP报文。

# kali网络配置
由于我使用的是校园网，因此我这里使用NAT模式。
{% asset_img p1.png p1 %} 

## 1.查看主机地址

首先通过`win`+`R`输入cmd打开终端，然后通过`ipconfig`查看自己主机的ip地址，网关，网段（我这里选择VMnet8）。
{% asset_img p2.png p2 %} 

```
IPv4 地址: 192.168.253.1
子网掩码: 255.255.255.0
```

## 2.修改/etc/network/interfaces文件
然后，通过`vim /etc/network/interfaces`，设置静态IP：
address：把ip地址设置为自己主机网段中的一个，如：我的主机ip是192.168.253.1，这里设为192.168.253.36
gateway：在其中加上自己主机的网关，如：192.168.253.2
netmask：子网掩码照抄自己主机的，如：255.255.255.0

```
auto eth0
iface eth0 inet static
address 192.168.253.36
netmask 255.255.255.0
gateway 192.168.253.2
```

## 3.修改dns
`vim /etc/resolv.conf`添加几个常用DNS：

```
domain localdomain
search localdomain
nameserver 8.8.8.8
nameserver 114.114.114.114
nameserver 192.168.253.2
```

## 4.重启网络
```
systemctl restart networking
```
或者
```
/etc/init.d/networking restart
```

## 5.测试
输入`ifconfig -a`查看ip：
{% asset_img p3.png p3 %} 


## nmap参数速查


| 命令 | 描述 |
| :------: | :------: |
|nmap IP|	扫描IP|
|nmap -v IP|	加强扫描|
|nmap IP1 IP2 ...|	扫描多IP|
|nmap a.b.c.*|	扫描整个子网|
|nmap a.b.c.x,y,...|	扫描多子网地址|
|nmap -iL xxx.txt|	根据文件扫描多IP|
|nmap a.b.c.x-y|	扫描子网IP范围|
|nmap a.b.c.* --exclude IP|	排除指定IP扫描整个子网|
|nmap -A IP|	全面的系统扫描,包括操作系统探测、版本探测、脚本扫描、路径跟踪|
|nmap -O IP|	探测操作系统|
|nmap -sA/-PN IP|	探测防火墙|
|nmap -sP a.b.c.*| 探测在线主机|
|nmap -F IP|	快速扫描|
|nmap -r IP|	按顺序扫描|
|nmap -iflist|	显示接口和路由信息|
|nmap -p n1,n2... IP|	扫描指定端口|
|nmap -p T:n1,n2... IP|	扫描TCP端口|
|nmap -sU n1,n2... IP|	扫描UDP端口|
|nmap -sV IP|	查看服务的版本|
|nmap -PS IP|	TCP ACK扫描|
|nmap -PA IP|	TCP SYN扫描|
|nmap -sS IP|	隐蔽扫描|
|nmap -sN IP|	TCP空扫描欺骗防火墙|

# 主机扫描
> 开两个虚拟机：192.168.253.36 + 192.168.253.37

## 主机扫描参数
主机发现有时候也叫做ping扫描，有以下选项：
`-sL`（列表扫描）：它仅仅列出指定网络上的每台主机，不发送任何报文到目标主机；
`-sP`（Ping扫描）：该选项告诉Nmap仅仅进行ping扫描（主机发现），然后打印出对扫描做出响应的那些主机；Nmap会发一个ICMP ECHO请求和一个TCP报文到目标端口；
`-Pn`（无ping）：无Ping扫描通常用于防火墙禁止Ping的情况下，完全跳过Nmap发现阶段，且对目标主机进行端口扫描；
`-PS`（TCP SYN ping）：该选项发送了一个设置了SYN标志位的空TCP报文，需要root权限；
`-PA`（TCP ACK ping）：它与TCP SYN Ping扫描类似，不同的只是设置的TCP报文的标志位是ACK；
`-PU`（UDP  ping）：发送一个空的UDP报文到指定端口，很少用；
`-PE/PP/PM`（ICMP ECHO/时间戳/地址掩码Ping）；
`-ARP`（ARP ping）：该选项通常在扫描局域网时使用，在内网中使用ARP Ping是非常有效的。

## 实验过程
```bash
nmap -P 192.168.253.1/24
```
{% asset_img p4.png p4 %} 

wireshark抓取ARP请求和响应报文：
{% asset_img p5.png p5 %}

ping命令发起ICMP扫描，wireshark抓取ICMP报文：
{% asset_img p6.png p6 %}

# 端口扫描
## 端口扫描基础
### 端口状态
Nmap所识别的6个端口状态：
- `Opend`：端口开启；
- `Closed`： 端口关闭；
- `Filtered`：端口被过滤，数据没有到达主机，返回的结果为空，数据被防火墙拦截了；
- `Unfiltered`：未被过滤，数据有到达主机，但是不能识别端口的当前状态；
- `Open|filtered`：开放或者被过滤，端口没有返回值，主要发生在UDP、IP、FIN、NULL和X mas扫描中；
- `Closed|filtered`：关闭或者被过滤，只发生在IP ID idle扫描。

### 命令参数
-`sS`（TCP SYN扫描）：半开扫描，很少有系统能把它记入系统日志。不过，需要Root权限；
-`sT`（TCP connect()扫描）:当SYN扫描不能用时，CP Connect()扫描就是默认的TCP扫描；
-`sN/-sF/-sX`（TCP Null/FIN/Xmas扫描）:能躲过一些无状态防火墙和报文过滤路由器；
-`p`：指定范围性扫描端口；
-`v`：详细信息。

## 实验过程

``` bash
┌──(root㉿kali)-[~]
└─# nmap -sS 192.168.253.1
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-07 22:42 EDT
Nmap scan report for 192.168.253.1
Host is up (0.00019s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
4001/tcp open  newoak
MAC Address: 00:50:56:C0:00:08 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 4.70 seconds
```

打开wireshark抓取tcp包，可看到：
{% asset_img p7.png p7 %}
扫描主机的135和139端口已经回发数据了。