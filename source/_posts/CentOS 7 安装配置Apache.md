---
title: CentOS 7 安装配置Apache
tags: Linux
categories: Linux
abbrlink: 13790
date: 2018-11-26 09:49:23
---

## 安装
```
sudo yum install httpd
```
<!-- more -->
## 添加网站
新建目录`/var/www/demo`
将发布好的网站粘贴到该目录

## 修改配置
```
vi /etc/httpd/conf/httpd.conf
```
假设网站的端口为8100,
```
Listen 8100

NameVirtualHost *:8100
<VirtualHost *:8100>
ServerName demo
DocumentRoot /var/www/demo
</VirtualHost>
```

## 配置semanage
安装 `semanage`
```
yum install policycoreutils-python
```
查看现在的支持http的端口有哪些
```
semanage port -l|grep http
```
为http服务添加新的端8100
```
semanage port -a -t http_port_t -p tcp 8100
```
## 防火墙开通8100端口
```
 firewall-cmd --zone=public --add-port=8100/tcp --permanent
 firewall-cmd --reload
```
## 其他命令
```
# 开机自启
sudo systemctl enable httpd
# 启动
sudo systemctl start httpd
# 停止
sudo systemctl stop httpd
```