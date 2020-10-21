---
title: Jenkins使用webhooks
tags: 教程
categories: jenkins
date: 2020-10-21 13:53:23
---

jenkins原生支持webhoos触发，无需安装任何插件，但是有权限验证，所以需要给用户生成一个token，
触发地址为：
```
http://username:token@jenkinshost:port/job/任务名称/build
```