---
title: windows设置和删除代理
date: 2018-05-03 11:07:38
updated: 2018-05-03 11:07:38
categories: 工具
tags:
  - windows
  - cmd
  - gitBash
  - http_proxy
---

起因是我在使用`ngrok` 的时候提示
`Invalid configuration property value for 'http_proxy', '******@qq.com'`

在 gitBash 使用
`vue init webpack my-project`
`gem install jekyll`
类似各种语言&框架的安装程序均不能正确安装
<!-- more -->
## 临时生效
  命令行窗口关闭就失效
  1. 设置 
  cmd: `set http_proxy='代理服务器地址:代理服务器端口'`

  gitBash: `export  http_proxy=ss`

  如果需要账号密码
  cmd:
  `set http_proxy_user=用户名`
  `set http_proxy_pass=密码`

  gitBash:
  `export  http_proxy_user=用户名`
  `export  http_proxy_pass=密码`

  2. 查看
  cmd: `set http_proxy`
  gitBash: `export -p`

  3. 删除
  cmd:`set http_proxy=`
  gitBash: `export -n http_proxy`

  >其实并没有什么卵用,因为窗口关闭就没有了

## 永久生效
  关机重启也不会生效
  `wmic`命令在cmd和gitBash通用
  1. 设置 
  `wmic ENVIRONMENT create name="http_proxy",username="<system>", VariableValue="代理服务器地址:代理服务器端口"`
  2. 查看
  `wmic ENVIRONMENT where "name='http_proxy'" get UserName,VariableValue`
  3. 删除
  `wmic ENVIRONMENT where "name='http_proxy'" delete`