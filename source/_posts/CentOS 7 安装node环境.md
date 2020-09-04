---
title: CentOS 7 安装node环境
tags: Linux
categories: Linux
abbrlink: 5588
date: 2018-11-23 09:49:23
---
### 下载
假设存放的目录为`/usr/software`
```
cd /usr/software
wget https://nodejs.org/dist/v10.9.0/node-v10.9.0-linux-x64.tar.xz  // 下载
tar xf  node-v10.9.0-linux-x64.tar.xz  // 解压
cd node-v10.9.0-linux-x64/bin  // 进入解压目录
node -v //查看node版本号
```
<!-- more -->
### 设置环境变量
```
vi /etc/profile
```
在文档末尾添加(目录要以node下载的目录为准)
```
export PATH=$PATH:/usr/software/node-v10.9.0-linux-x64/bin
```
配置生效
```
source /etc/profile
```