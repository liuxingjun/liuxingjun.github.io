---
title: grafana自定义告警模版
date: 2022-03-16 10:34:00
updated: 2022-03-26 20:50:00
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

## 方式一： 文本类告警
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
## 方式二： 邮箱告警
如果使用邮箱告警的情况下修改 contact point 的 Message  使用自定义邮箱模版会导致界面太丑，没有样式的问题

在官网文档内 [HTML in Message Templates](https://grafana.com/docs/grafana/latest/alerting/unified-alerting/message-templating/#html-in-message-templates) 修改邮箱告警模版需要修改`ng_alert_notification.html`文件

编辑`public/emails/ng_alert_notification.html` 文件  
如果要删除Value字段，则删除[213-217行](https://github.com/grafana/grafana/blob/v8.4.3/public/emails/ng_alert_notification.html#L213-L217)   
如果要删除Labels字段，则删除[227-L234行](https://github.com/grafana/grafana/blob/v8.4.3/public/emails/ng_alert_notification.html#L227-L234)  
如果是本地直接安装修改文件重启即可  
如果是docker or k8s 安装，需要将修改后的文件挂载到容器内  
2. 完成，等待收到告警消息
效果图![emailAlert.png](emailAlert.png)

