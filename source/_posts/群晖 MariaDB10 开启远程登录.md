---
title: 群晖 MariaDB10 开启远程登录
tags: 群晖
categories: 教程
date: 2020-11-03 13:53:23
---

群晖安装 MariaDB10 后，默认仅支持本机连接，也就是说，你的局域网电脑是连接不上的，如果需要局域网连接，需要做处理。
首先使用 ssh 连接到群晖，进入 MariaDB 默认安装目录
```
cd /volume1/@appstore/MariaDB10/usr/local/mariadb10/bin
```
使用 root 登录 MariaDB，输入密码
```
./mysql -u root -p
```
切换 mysql 数据库
```
MariaDB [(none)]> use mysql
```
更新user表
```
MariaDB [mysql]> update user set host = '%' where user = 'root';
```
刷新权限
```
MariaDB [mysql]> FLUSH PRIVILEGES;
```