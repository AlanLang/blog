---
title: CentOS 7一键安装Seafile搭建私有云存储
tags: Linux
categories: Linux
abbrlink: 21644
date: 2018-03-15 12:49:23
---
[参考手册](https://manual-cn-origin.seafile.com)
## 环境要求
* CentOS 7 64位
* Python >= 2.7
* SqLite 3

<!-- more -->
## 开始安装
复制下面的命令，依次输入，然后按照提示进行安装即可
```
yum -y install wget
wget https://raw.githubusercontent.com/helloxz/seafile/master/install_seafile.sh
chmod +x install_seafile.sh && ./install_seafile.sh
```
## 配置邮件发送
邮件提醒会使某些功能有更好的用户体验, 比如发送邮件提醒用户新消息到达. 请在seahub_settings.py中加入以下语句以开启邮件提醒功能 (同时需要对你的邮箱进行设置).
Gmail 邮箱示例:
```
EMAIL_USE_TLS = True
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = 'username@gmail.com'
EMAIL_HOST_PASSWORD = 'password'
EMAIL_PORT = '587'
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
SERVER_EMAIL = EMAIL_HOST_USER
```
QQ 邮箱示例：
```
EMAIL_USE_SSL = True
EMAIL_HOST = 'smtp.qq.com'
EMAIL_HOST_USER = 'username@domain.com'
EMAIL_HOST_PASSWORD = 'Auth_Code'
EMAIL_PORT = '465'
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
SERVER_EMAIL = EMAIL_HOST_USER
```
## 其它说明
安装目录和服务如下，如果您还需要配置更多的功能或扩展，请访问官方帮助文档：https://manual-cn.seafile.com/
```
#安装目录
/home/MyCloud
#启动服务
/home/MyCloud/seafile-server/seafile.sh start
/home/MyCloud/seafile-server/seahub.sh start
#停止服务
/home/MyCloud/seafile-server/seafile.sh stop
/home/MyCloud/seafile-server/seahub.sh stop
```