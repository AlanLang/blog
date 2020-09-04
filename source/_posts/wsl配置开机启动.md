---
title: wsl配置开机启动
tags: linux
categories: linux
abbrlink: 59040
date: 2018-12-24 09:49:23
---
以`ubuntu`配置开机启动`ssh`为例。
### 免密码使用`sudo`直接使用`root`权限执行命令
vi /etc/sudoers
<!-- more -->

```
alan_ubuntu ALL=(ALL) NOPASSWD: /usr/sbin/servic
```
### 在windows下新建`wslstartup.vbs`
```
set ws=wscript.createobject("wscript.shell")
ws.run "C:\Windows\System32\bash.exe",0
ws.run "C:\Windows\System32\bash.exe  -c 'sudo /usr/sbin/service ssh start'",0
```
###将 `wslstartup.vbs`加入计划任务
![](http://dada-image-bed.oss-cn-shenzhen.aliyuncs.com/18-12-24/94766517.jpg)
![](http://dada-image-bed.oss-cn-shenzhen.aliyuncs.com/18-12-24/9969189.jpg)
![](http://dada-image-bed.oss-cn-shenzhen.aliyuncs.com/18-12-24/23785132.jpg)