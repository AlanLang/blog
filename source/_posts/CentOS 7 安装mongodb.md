---
title: CentOS 7 安装mongodb
tags: Linux
categories: Linux
abbrlink: 3512
date: 2018-11-23 13:49:23
---
### 下载
假设存放的目录为`/usr/software`
```
cd /usr/software
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.6.tgz  // 下载
tar -zxvf mongodb-linux-x86_64-3.0.6.tgz  // 解压
```
<!-- more -->
### 设置环境变量
```
vi /etc/profile
```
在文档末尾添加(目录要以node下载的目录为准)
```
export PATH=$PATH:/usr/software/mongodb-linux-x86_64-3.0.6/bin
```
配置生效
```
source /etc/profile
```
### 配置并运行
接着在usr/software/mongodb目录下新建一个名为mongodb.conf的配置文件，写入如下配置内容
```
port=27017
dbpath=/usr/software/mongodb/data/db
logappend=true
fork=true
logpath=/usr/software/mongodb/data/logs
```
保存。然后输入命令启动mongod --config /usr/software/mongodb/mongodb.conf

### 开机自启
```
cd /lib/systemd/system/
vi mongodb.service
```
添加内容
```
[Unit]
Description=mongodb
After=network.target remote-fs.target nss-lookup.target
[Service]
Type=forking
ExecStart=/usr/software/mongodb-linux-x86_64-3.0.6/bin/mongod --config /usr/software/mongodb/mongodb.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/usr/software/mongodb-linux-x86_64-3.0.6/bin/mongod --shutdown --config /usr/software/mongodb/mongodb.conf
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```
配置生效
```
systemctl daemon-reload
```
可用命令
```
#启动服务
systemctl start mongodb.service
#关闭服务
systemctl stop mongodb.service
#开机启动
systemctl enable mongodb.service
```