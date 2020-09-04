---
title: CentOS 7 修改yum源
tags: Linux
categories: Linux
date: 2018-03-16 9:49:23
abbrlink: 61758
---
备份本地yum源
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo_bak 
```
获取阿里yum源配置文件
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
```
更新cache
```
yum makecache 
```
查看
```
yum -y update 
```