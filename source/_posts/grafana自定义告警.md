---
title: grafana自定义告警模版
date: 2022-03-16 10:34:00
updated: 2022-03-16 10:34:00
categories:
tags:
  - grafana
---

因 grafana 告警信息太多无用数据，容易干扰查看
例如使用 企业微信告警消息如下
![alert.png](alert.png)
太多无用Labels 例如 endpoint,job,pod,service 
还有Value 字段太占空间,Source, Silence 字段,因太长且无用
所以修改模版来精简告警消息

## 方式一： 使用模版
例如企业微信类的文本告警，根据官网文档知道默认模版来自 [default_template.go](https://github.com/grafana/grafana/blob/main/pkg/services/ngalert/notifier/channels/default_template.go) 
主要定义了2个模版，一个是`__text_alert_list` ，另一个`default.message` 使用了模版`__text_alert_list`
我们只需要根据`__text_alert_list` 模版修改为自定义模版即可
1. 创建一个消息告警模版
打开grafana 页面，点击左侧 alerting 图标 ，点击 Contact points 菜单 ，点击New template 按钮
Template name 填写 text_alert_list
Content 内 复制如下原始模版
<!-- more -->
```
{{ define "__text_alert_list" }}{{ range . }}
Value: {{ or .ValueString "[no value]" }}
Labels:
{{ range .Labels.SortedPairs }} - {{ .Name }} = {{ .Value }}
{{ end }}Annotations:
{{ range .Annotations.SortedPairs }} - {{ .Name }} = {{ .Value }}
{{ end }}{{ if gt (len .GeneratorURL) 0 }}Source: {{ .GeneratorURL }}
{{ end }}{{ if gt (len .SilenceURL) 0 }}Silence: {{ .SilenceURL }}
{{ end }}{{ if gt (len .DashboardURL) 0 }}Dashboard: {{ .DashboardURL }}
{{ end }}{{ if gt (len .PanelURL) 0 }}Panel: {{ .PanelURL }}
{{ end }}{{ end }}{{ end }}
```
修改`__text_alert_list` 为 `text_alert_list`
删除 自己认为多余的信息，例如Value 和Labels ，记得需要成对删除，否则可能导致模版不可用
如下是我修改的模版
```
{{ define "text_alert_list" }}{{ range . }}
Annotations:
{{ range .Annotations.SortedPairs }} - {{ .Name }} = {{ .Value }}
{{ end }}{{ end }}{{ end }}
```
2. 新建一个模版支持整个通知信息
Template name 填写 message
Content 填写
```
{{ define "message" }}{{ if gt (len .Alerts.Firing) 0 }}**Firing**
{{ template "text_alert_list" .Alerts.Firing }}{{ if gt (len .Alerts.Resolved) 0 }}

{{ end }}{{ end }}{{ if gt (len .Alerts.Resolved) 0 }}**Resolved**
{{ template "text_alert_list" .Alerts.Resolved }}{{ end }}{{ end }}
```
最终如下
![template.png](template.png)
3. 修改企业微信的 contact point
在Optional WeCom settings 下 的 Message 输入框内填写
```
{{ template "message" . }}
```
这里的`message` 对应上一步的`define "message"` 
4. 完成，等待收到告警消息
效果图
![wechatAlert.png](wechatAlert.png)
## 方式二： 直接修改contact point 内的Message 字段
此方式适合邮箱告警
在官网文档内 [HTML in Message Templates](https://grafana.com/docs/grafana/latest/alerting/unified-alerting/message-templating/#html-in-message-templates) 
修改邮箱告警模版需要修改`ng_alert_notification.html`文件
但是helm 安装修改比较麻烦，所以邮箱告警使用此方式
!!#ff0000 ！！！无法在企业微信告警内使用此方法!!
1. 修改邮箱告警的 contact point
在Optional WeCom settings 下 的 Message 输入框内填写
```
{{ range . }}
Annotations:
{{ range .Annotations.SortedPairs }} - {{ .Name }} = {{ .Value }}
{{ end }}
```
2. 完成，等待收到告警消息
效果图![emailAlert.png](emailAlert.png)

