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
> Kali：是一个基于Debian的Linux发行版，预装了许多渗透测试软件，是一个很酷的“黑客操作系统”
> Wireshark：是一个网络封包分析软件，功能是截取网络封包，并尽可能显示出最为详细的网络封包资料。Wireshark使用WinPCAP作为接口，直接与网卡进行数据报文交换
> Nmap：即Network Mapper，是Linux下的网络扫描和嗅探工具包

# 实验内容
1.在Nmap中，使用nmap –sP xx指令实现主机扫描，用wireshark抓取扫描过程中的ARP请求和响应报文；通过ping指令发起ICMP扫描，用wireshark抓取扫描过程中的ICMP报文。
2.在Nmap中，使用nmap –sS xx指令实现端口扫描，用wireshark抓取扫描过程中的TCP报文。

# kali网络配置
由于我使用的是校园网，因此我这里使用NAT模式。
{% asset_img p1.png p1 %} 

1.查看主机地址

首先通过`win`+`R`输入cmd打开终端，然后通过`ipconfig`查看自己主机的ip地址，网关，网段（我这里选择VMnet8）。
{% asset_img p2.png p2 %} 

```
IPv4 地址: 192.168.253.1
子网掩码: 255.255.255.0
```

2.修改/etc/network/interfaces文件
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

3.修改dns
`vim /etc/resolv.conf`添加几个常用DNS：

```
domain localdomain
search localdomain
nameserver 8.8.8.8
nameserver 114.114.114.114
nameserver 192.168.253.2
```

4.重启网络
```
systemctl restart networking

```
或者
```
/etc/init.d/networking restart
```

5.测试
输入`ifconfig -a`查看ip：
{% asset_img p3.png p3 %} 

# 主机扫描
> 开两个虚拟机：192.168.253.36 + 192.168.253.37
主机发现有时候也叫做ping扫描，有以下选项：
-sL(列表扫描)：它仅仅列出指定网络上的每台主机，不发送任何报文到目标主机
-sP(Ping扫描)：该选项告诉Nmap仅仅进行ping扫描(主机发现)，然后打印出对扫描做出响应的那些主机
-P0(无ping)：完全跳过Nmap发现阶段
-PS/PA/PU/PE/ARP[portlist]：使用TCP SYN/ACK、UDP、ICMP ECHO、ARP方式进行发现

```bash
nmap -P 192.168.253.1/24
```
{% asset_img p4.png p4 %} 

wireshark抓取ARP请求和响应报文：
{% asset_img p5.png p5 %}

ping命令发起ICMP扫描，wireshark抓取ICMP报文：
{% asset_img p6.png p6 %}

# 端口扫描
