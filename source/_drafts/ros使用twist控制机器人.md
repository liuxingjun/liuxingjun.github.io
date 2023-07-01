---
title: ros使用twist控制机器人
date: 2023-05-04 10:55
updated: 2023-05-04 10:55
categories: 
tags: 
---
## 消息结构

使用ros2 msg [Twist](https://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Twist.html)

linear 代表线性， 其中的x 用来控制机器人前进和后退，前进为正，后退为负,值的大小代表速度
angular 代表角度，其中的z 用来控制机器人转向, 左转为正，右转为负,值的大小代表角度的大小

示例

```
linear:
  x: 0.5
angular:
  z: 0.0
```
表示已0.5的速度前进

```
linear:
  x: -1
angular:
  z: 0.0
```
表示已1的速度后退

```
linear:
  x: 1
angular:
  z: 0.5
```
表示以1的速度，0.5的角度向左前方 前进

```
linear:
  x: 1
angular:
  z: -0.5
```
表示以1的速度，0.5的角度向右前方 前进

```
linear:
  x: -1
angular:
  z: 0.5
```
表示以1的速度，0.5的角度向右后方 后退

