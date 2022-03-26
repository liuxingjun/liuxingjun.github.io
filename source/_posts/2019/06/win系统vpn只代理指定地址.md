---
title: win系统vpn只代理指定地址
date: 2019-06-17 16:48:25
updated: 2019-06-17 16:48:25
categories: 效率
tags:
  - windows
  - vpn
---
添加一个 vpn

假设我的vpn名称为`220.***.***.64`

取消勾选 `在远程网络上使用默认网关` 

查看需要访问的网站的ip
```
PS C:\Users\Administrator> ping www.google.com

正在 Ping www.google.com [69.171.248.65] 具有 32 字节的数据:
```

`69.171.248.65` 即需要访问的ip 

<!-- more -->

查看 vpn 分派的IP地址和子网掩码
```
 C:\Users\Administrator> ipconfig
Windows IP 配置
以太网适配器 vEthernet (默认开关):
   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::9851:3dc9:7749:8e7a%14
   IPv4 地址 . . . . . . . . . . . . : 172.17.130.113
   子网掩码  . . . . . . . . . . . . : 255.255.255.240
   默认网关. . . . . . . . . . . . . :
以太网适配器 以太网:
   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::803d:b15a:a6b3:23af%5
   IPv4 地址 . . . . . . . . . . . . : 192.168.0.207
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . : 192.168.0.1
PPP 适配器 220.***.***.64:  
   连接特定的 DNS 后缀 . . . . . . . :
   IPv4 地址 . . . . . . . . . . . . : 172.30.28.12 (这里是vpn分配的IP)
   子网掩码  . . . . . . . . . . . . : 255.255.255.255 (这里是vpn的子网掩码)
   默认网关. . . . . . . . . . . . . :
```

单次生效
`route add 需要访问的ip  mask  vpn的子网掩码 vpn的ip` 

永久生效
`route add -p 需要访问的ip  mask  vpn的子网掩码 vpn的ip` 

更新(2019-07-02)

vpn 内网IP非固定导致断连后 IP 变动 路由失效

动态获取 vpn 内网 IP 添加到路由

```
 @echo off

for /f "delims=: tokens=2" %%i in ('netsh interface ip show config name^=220.168.85.64 ^| findstr "IP Address"') do (set gateway=%%i)

ECHO %gateway%

route delete 69.171.248.65
route add 69.171.248.65 mask 255.255.255.255 %gateway%

pause>nul
```




