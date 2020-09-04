---
title: EF 知识收集与问题解决
tags: C#
categories: 知识点
abbrlink: 25896
date: 2018-06-25 14:24:34
---
## 语法学习
[用linqPad帮助你快速学习LINQ](https://www.cnblogs.com/li-peng/p/3441729.html)
### GroupBy
```
var re = db.InvoiceDetail.WhereDynamic(Request.Form).GroupBy(a => new { a.IvdMatCode, a.IvdMatName }).Select(a => new
{
    IvdMatCode = a.Key.IvdMatCode,
    IvdMatName = a.Key.IvdMatName,
    num = a.Sum(x => x.IvdScanNum)
}).ToList();
```
<!-- more -->
------

## 知识收集
### 数据库迁移
打开迁移：`Enable-Migrations`
增加迁移的节点：`Add-Migration PaperTest`
将修改升级到数据库:`Update-Database`
------

## 问题解决
### Update-Database : 无法将“Update-Database”项识别为 cmdlet、函数、脚本文件或可运行程序的名称的问题
原因：
没有引用EntityFramework命令
解决：
在程序包管理器控制台执行如下命令:
Import-Module 项目路径\packages\EntityFramework.6.1.3（EF版本）\tools\EntityFramework.psd1
------