---
title: CentOS 7 防火墙操作
tags: Linux
categories: Linux
abbrlink: 26003
date: 2018-11-24 09:49:23
---

## 基本使用
* 启动： `systemctl start firewalld`
* 关闭： `systemctl stop firewalld`
* 查看状态： `systemctl status firewalld`
* 开机禁用  ： `systemctl disable firewalld`
* 开机启用  ： `systemctl enable firewalld`
<!-- more -->

## 开放端口
### 添加
```
firewall-cmd --zone=public --add-port=80/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）
```
### 重新载入
```
firewall-cmd --reload
```
### 查看
```
firewall-cmd --zone= public --query-port=80/tcp
```
### 删除
```
firewall-cmd --zone= public --remove-port=80/tcp --permanent
```