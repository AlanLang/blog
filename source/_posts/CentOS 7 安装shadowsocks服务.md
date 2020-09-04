---
title: CentOS 7 安装shadowsocks服务
tags: Linux
categories: Linux
abbrlink: 17613
date: 2018-11-23 09:49:23
---

### 安装pip
`Pip`是`Python`的包管理工具，下载`ss`十分方便，但是`centos`是没有`pip`的，我们需要安装安装一个。
```
yum install python-setuptools & easy_install pip
```
<!-- more -->
### 安装Shadowsocks 客户端
```
pip install shadowsocks
```
修改配置文件
```
vi /etc/shadowsocks.json
```
进入编辑模式，粘贴以下内容
```
{
  "server": "0.0.0.0",
  "port_password": {
    "此处填写端口号": "此处填写密码"
  },
  "timeout": 300,
  "method": "aes-256-cfb",
  "fast_open": false,
  "workers": 1
}
```
### 启动 shadowsocks
```
# 启动
ssserver -c /etc/shadowsocks.json -d start
# 停止
ssserver -c /etc/shadowsocks.json -d stop
# 重新启动
ssserver -c /etc/shadowsocks.json -d restart
```
查看是否正常启动
```
ps -aux | grep ssserver
```
### 设置自启动
编辑`rc.local`文件
```
vi /etc/rc.local
```
添加
```
sudo ssserver -c /etc/shadowsocks/config.json -d start
```