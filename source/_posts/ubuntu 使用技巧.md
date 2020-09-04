---
title: ubuntu 使用技巧
tags: ubuntu
abbrlink: 59562
date: 2019-07-09 12:00:34
---

## 快速安装最新版的node.js
### 去 nodejs 官网 https://nodejs.org 看最新的版本号
例如现在最新的版本号是12.6.0
### 添加源并安装
nodejs 的每个大版本号都有相对应的源，比如这里的 12.x.x版本的源是https://deb.nodesource.com/setup_12.x。

所以在终端执行： 
```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
稍等片刻，源已经添加完毕，再执行：
``` 
sudo apt-get install -y nodejs
```
等待安装完成。

<!-- more -->

## 防火墙操作
打开或关闭某个端口，例如：
```
sudo ufw allow smtp　允许所有的外部IP访问本机的25/tcp (smtp)端口
sudo ufw allow 22/tcp 允许所有的外部IP访问本机的22/tcp (ssh)端口
sudo ufw allow 53 允许外部访问53端口(tcp/udp)
sudo ufw allow from 192.168.1.100 允许此IP访问所有的本机端口
sudo ufw allow proto udp 192.168.0.1 port 53 to 192.168.0.2 port 53
sudo ufw deny smtp 禁止外部访问smtp服务
sudo ufw delete allow smtp 删除上面建立的某条规则
```
查看防火墙状态
```
sudo ufw status
```

## 安装fish
```
sudo apt-add-repository ppa:fish-shell/release-3
sudo apt-get update
sudo apt-get install fish
```

## 安装jenkins
### 安装 open-jdk
```
sudo apt-get install openjdk-8-jdk
```
### 安装jenkins
将存储库密钥添加到系统。
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
```
我们将Debian包存储库地址附加到服务器的`sources.list`
```
echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
```
更新存储库
```
sudo apt-get update
```
安装`Jenkins`
```
sudo apt-get install jenkins
```
安装完成后，jenkins的文件目录如下所示
```
安装目录：/var/lib/jenkins  
日志目录：/var/log/jenkins/jenkins.log  
```
### 修改端口号
```
vi /etc/default/jenkins
```
### 启动
```
service jenkins start 
```