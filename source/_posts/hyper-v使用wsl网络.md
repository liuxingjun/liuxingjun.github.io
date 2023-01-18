---
title: 在内网环境下hyper-v使用wsl网络
date: 2023-01-18 16:30
updated: 2023-01-18 16:30
categories: windows 
tags:
  - wsl
  - hyper-v
---

# 前言


# 环境
系统版本: Windows 11 专业版 22H2

WSL版本
```PowerShell
PS C:\Users\v_connliu> wsl -v
WSL 版本： 1.0.3.0
内核版本： 5.15.79.1
WSLg 版本： 1.0.47
MSRDC 版本： 1.2.3575
Direct3D 版本： 1.606.4
DXCore 版本： 10.0.25131.1002-220531-1700.rs-onecore-base2-hyp
Windows版本： 10.0.22621.1105
```

`.wslconfig`和`/etc/wsl.conf`均使用默认配置


## 目前方案和需求

本人在一个内网环境,使用登录公司特定软件才可以访问网络
使用一台windows11电脑上,使用wsl开启的linux 环境用于日常linux 开发使用
因开发或者其他需要使用一个vpn代理软件,但因公司安全策略不能在宿主机安装,只好使用hyper-v 安装一个虚拟机在里面使用

使用hyper-v时网络是最大的难题
### 使用Default Switch
在hyper-v内使用Default Switch 没有网络 
遂放弃
### 使用WSL
查看wsl网关
```bash
root@ubuntu:~# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.31.176.1    0.0.0.0         UG    0      0        0 eth0
172.31.176.0    0.0.0.0         255.255.240.0   U     0      0        0 eth0`
```
进入虚拟机后修改ip,子网掩码,网关,dns服务器
ip:  使用 172.31.176.0 同网段
子网掩码:255.255.240.0
网关: 172.31.176.1
dns服务器: 172.31.176.1
这样虚拟机就可以正常上网了, 但是 wsl 重启会导致 网关变动,进而导致 使用固定网关的虚拟机内也无法使用网络

需要一种解决方案既可以在wsl 可以访问网络,在 虚拟机内也可以访问网络的方案

## 固定wsl 的Gateway

修改`/etc/wsl.conf`的systemd 配置
```ini
[boot]
systemd=true
```

修改`/etc/systemd/network/eth0.network`为如下内容
```ini /etc/systemd/network/eth0.network
[Match]
Name=eth0

[Network]
Address=172.17.112.46/20
Gateway=172.17.112.1
Description=lan
DHCP=no
EmitLLDP=true
LLDP=true
IPForward=true
```

重启生效
```PowerShell 
wsl --shutdown
```