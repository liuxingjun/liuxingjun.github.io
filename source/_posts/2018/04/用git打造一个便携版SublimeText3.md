---
title: 用git打造一个便携版SublimeText3
date: 2018-04-27 11:59:02
updated: 2018-04-27 11:59:02
categories: 工具
tags:
  - git
  - sublime text 3
---

## 下载 & 安装
  官网已经提供便携版本(portable version)下载了, 也可以点击此处下载
  
  [32bit](https://download.sublimetext.com/Sublime%20Text%20Build%203143.zip)

  [64bit](https://download.sublimetext.com/Sublime%20Text%20Build%203143%20x64.zip)

  解压到安装目录
  
## 添加右键菜单

  新建一个后缀为`.inf`的文件，内容为
<!-- more -->
  ```
  [Version]
  Signature="$Windows NT$"

  [DefaultInstall]
  AddReg=SublimeText3

  [SublimeText3]
  hkcr,"*\\shell\\SublimeText3",,,"ÓÃ SublimeText3 ´ò¿ª"
  hkcr,"*\\shell\\SublimeText3\\command",,,"""%1%\sublime_text.exe"" ""%%1"" %%*"
  hkcr,"*\\shell\\SublimeText3","Icon",0x20000,"%1%\sublime_text.exe, 0"

  hkcr,"Directory\shell\SublimeText3",,,"ÓÃ SublimeText3 ´ò¿ª"
  hkcr,"Directory\shell\SublimeText3\command",,,"""%1%\sublime_text.exe"" ""%%1"""
  hkcr,"Directory\shell\SublimeText3","Icon",0x20000,"%1%\sublime_text.exe, 0"
  ```

  右键点击文件，选择安装
## 注册
  如果用的我的链接下载的为3143版，可直接使用以下LICENSE，未通过其他版本测试
  ```
  —– BEGIN LICENSE —–  
  TwitterInc  
  200 User License  
  EA7E-890007  
  1D77F72E 390CDD93 4DCBA022 FAF60790  
  61AA12C0 A37081C5 D0316412 4584D136  
  94D7F7D4 95BC8C1C 527DA828 560BB037  
  D1EDDD8C AE7B379F 50C9D69D B35179EF  
  2FE898C4 8E4277A8 555CE714 E1FB0E43  
  D5D52613 C3D12E98 BC49967F 7652EED2  
  9D2D2E61 67610860 6D338B72 5CF95C69  
  E36B85CC 84991F19 7575D828 470A92AB  
  —— END LICENSE ——  
  ```

  如果有注册成功后 
## 上传github
  .gitignore
  ```
  Date/Cache #目前已知这个目录不影响使用，每次删除都可以重新生成
  ```
>以后安装了插件 或者修改了什么配置，再上传到Git即可

## 卸载
1. 删除右键菜单

  双击sublime_delright.reg卸载右键菜单

2. 删除当前目录

最后附上我自己的成果 [SublimeText3](https://github.com/liuxingjun/Sublime-Text-3)
