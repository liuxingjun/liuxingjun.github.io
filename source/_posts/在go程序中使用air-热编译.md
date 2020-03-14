---
title: 在go程序中使用热编译
date: 2020-03-14 11:04:46
categories: golang
tags: 
  - 热编译
---

我们在使用  [kratos](https://github.com/bilibili/kratos)  时 发现没有热部署，每次更新代码都要重新编译，并且重新执行
即使使用 `kratos run` 也要 停止命令，重新编译

第一个热编译选择了 [oxequa/realize](https://github.com/oxequa/realize)  ,实际使用中发现 各种无法运行 
可能是我打开的方式不对。而且项目好像已经停止维护了...

然后选择 使用 [cosmtrek/air](https://github.com/cosmtrek/air)   中乱码
```
[11:17:56] watching internal\service
[11:17:56] !exclude run
[11:17:56] watching test
[11:17:56] building...
[11:17:59] running...
'.' �����ڲ����ⲿ���Ҳ���ǿ����еĳ���
���������ļ���
[11:18:05] cleaning...
[11:18:05] see you again~
```
在所有win 命令行下均有此问题，包括cmd,GitBash,PowerShell
在win10 内置linux下无此问题
<!-- more -->
下面开始

1. 下载golang
 国内 golang 网址：[golang.google.cn](https://golang.google.cn/)  下载linux 压缩包, 可以使用浏览器下载
 在下载目录右键 `在此处打开linux shell` 
 解压文件 `tar -C /usr/local -xzf go1.14.linux-amd64.tar.gz` 
 增加环境变量 `export PATH=$PATH:/usr/local/go/bin`
 执行 `go version` 确认安装golang成功
2. 下载air
 浏览器下载 https://git.io/linux_air 移动到刚才命令行窗口的目录 执行命令  `mv air /usr/local/bin/air`
 或者  `cd /usr/local/bin/; wget -c -O air https://git.io/linux_air `
 执行 `air -v` 确认安装air成功
3. 使用air
 我是使用的kratos 
 ```smartyconfig
# Config file for [Air](https://github.com/cosmtrek/air) in TOML format

# Working directory
# . or absolute path, please note that the directories following must be under root.
root = "."
tmp_dir = "run"

[build]
# Just plain old shell command. You could use `make` as well.
#cmd = "go build -o run/main.exe main.go" #general
cmd = "go build -o run/cmd.exe ./cmd" #kratos 
# Binary file yields from `cmd`.
bin = "run/cmd.exe"
# Customize binary.
#full_bin = "./run/main.exe" #general
full_bin = "./run/cmd.exe -conf ./configs" #kratos 
# Watch these filename extensions.
include_ext = ["go", "tpl", "tmpl", "html"]
# Ignore these filename extensions or directories.
exclude_dir = [".git","vendor","run",".idea"]
# Watch these directories if you specified.
include_dir = []
# Exclude files.
exclude_file = [".air.conf"]
# It's not necessary to trigger build each time file changes if it's too frequent.
delay = 1000 # ms
# Stop to run old binary when build errors occur.
stop_on_error = true
# This log file places in your tmp_dir.
log = "air_errors.log"

[log]
# Show log time
time = true

[color]
# Customize each part's color. If no color found, use the raw app log.
main = "magenta"
watcher = "cyan"
build = "yellow"
runner = "green"

[misc]
# Delete tmp directory on exit
clean_on_exit = false
```
在刚才的窗口执行`air` 即可



