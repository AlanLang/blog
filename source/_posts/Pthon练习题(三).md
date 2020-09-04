---
title: Pthon练习题(三)
tags: python
categories: Pthon练习题
abbrlink: 8156ce4d
date: 2018-02-01 11:00:23
---
## 题目
将 0001 题生成的 200 个激活码（或者优惠券）保存到 **MySQL** 关系型数据库中。
<!-- more -->
## MySql学习
[MySql教程|菜鸟教程](http://www.runoob.com/mysql/mysql-tutorial.html)
## 安装MySql
```
yum install mysql
yum install mysql-devel
yum install mariadb-server mariadb
```
### 启动MySql
```
systemctl start mariadb.service
```
### 登陆MySql
```
mysql -u root -p
```
### 授权远程连接
```
mysql -u root -p
Enter password:
MariaDB [(none)]>
grant all privileges on *.* to 'root'@'192.168.*.*' identified by 'root' with grant option;
```
### 关闭防火墙
```
systemctl stop firewalld
```
## 新建表
使用Navicat for MySql 连接数据库，并在数据库`text`新建表`uuid_table`,效果如下：
![](http://oqdzx28cd.bkt.clouddn.com/18-1-9/21750120.jpg)

## 编码
### 安装pymysql3
```
pip3 install pymysql3
```
### demo.py
```Python
import uuid
import pymysql

def insertCode(codes):
	db = pymysql.connect(host='192.168.10.130', port=3306, user='root', passwd='123456',db='test')
	cursor = db.cursor()
	try:
		for code in codes:
			sql = "INSERT INTO uuid_table(uuid,datestr) VALUES ('%s', '%s')" % (code, '20180109')
			cursor.execute(sql)
			db.commit()
	except:
		db.rollback()
	db.close()

def generateCode(num):
	list = []
	for i in range(num):
		list.append(uuid.uuid1())
	return list

if __name__ == "__main__":
	codes = generateCode(200)
	insertCode(codes)
```
### 效果
![](http://oqdzx28cd.bkt.clouddn.com/18-1-9/95571570.jpg)
