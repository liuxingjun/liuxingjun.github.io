---
title: golang 管理多版本
date: 2022-11-08 17:53
updated: 2022-11-09 10:07
categories: golang
---

根据官方文档[manage-install](https://go.dev/doc/manage-install),我们使用go install 安装新版本golang

## 安装下载新版本
```bash
$ go install golang.org/dl/go1.18@latest
$ go1.18 download
$ go1.18 version
go version go1.18 linux/amd64
```
但是这个时候go1.18 并不是默认版本

## 设置新版的go1.18为默认 版本

which 得到程序的执行路径
```bash 
which gogo1.18
/root/go/bin/go1.18
```

使用`update-alternatives` 管理多版本go
`update-alternatives --install <软链> <名字> <路径> <优先级>`

```bash
update-alternatives --install /usr/bin/go go $(which go1.18) 900 
```

其他update-alternatives 命令
```bash
update-alternatives --display go #查看已安装
update-alternatives --auto go #自动按优先级(最大的数)设置版本 
update-alternatives --config go #修改版本
```

## 总结 

### 备份旧版

```bash
$ which go
/usr/local/go/bin/go
$ go version
go version go1.16 linux/amd64
$ mv /usr/local/go/bin/go /usr/local/go/bin/go1.16 
$ update-alternatives --install /usr/bin/go go $(which go1.16) 800 
```
### 一行命令安装新版golang
```bash
v=1.17 && go install golang.org/dl/go${v}@latest && go${v} download && update-alternatives --install /usr/bin/go go $(which go${v}) 900
```