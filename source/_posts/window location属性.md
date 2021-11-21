---
title: location对象
date: 2020/01/09
tags: ["location"]
---

### 属性

存放与当前 url 相关的信息

- hash 模式：（默认）
  http://192.168.1.97:3000/utregdsfghters/#/robot?projectId=10&proname=test7

- history 模式：
  http://192.168.1.97:3000/robot?projectId=10&proname=test7

- href：设置或返回完整的 URL
  —— http://192.168.1.97:3000/utregdsfghters/#/robot?projectId=10&proname=test7

- search：设置或返回从？开始的 URL（包含？）
  ——hash 模式下： ""
  ——history 模式下： ?projectId=10&proname=test7

- hash：设置或返回从#开始的 URL（包含#）
  —— hash 模式下：#/robot?projectId=10&proname=test7
  —— history 模式下：""

- pathname：获取端口号以后，？或#之前的部分
  —— /robot
  —— /utregdsfghters/

- host：设置或返回主机名和当前的 URL 端口号
  —— 192.168.1.97:3000

- hostname：设置或返回主机名称
  —— 192.168.1.97

- port: 设置或返回当前的端口号
  ——3000

- protocol：设置或返回当前 url 协议
  —— http:

### 方法

- assign():加载新的文档，必传参数：地址
- reload():重新加载当前文档
- replace():用新的文档替换当前文档，必传参数：地址，（被替换文档无历史记录）
