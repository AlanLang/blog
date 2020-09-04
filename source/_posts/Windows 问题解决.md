---
title: Windows 问题解决
tags: Windows
categories: 问题解决
abbrlink: 62820
date: 2018-07-22 12:00:34
---
## 简介
主要收集windows使用期间所遇到的问题以及解决方法。
<!-- more -->

## 问题描述
远程桌面登录时，出现身份验证错误，要求的函数不正确，这可能是由于CredSSP加密Oracle修正。
## 解决
解决方法1
运行 gpedit.msc 本地组策略，“计算机配置”->“管理模板”->“系统”->“凭据分配”但是我的却找不到“加密Oracle修正”选项，选择启用并选择易受攻击。

解决方法2
* 运行 regedit
* 打开注册表
* HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\CredSSP\Parameters
* 在 System（之后没有的文件夹，需自己创建）然后在最底部文件夹Parameters里面
* 新建 DWORD（32）位值（D）。文件名 “AllowEncryptionOracle” ，值 : 2，
* 保存，重启

以上2种方法都试了还不行，控制面板->所有控制面板项->Windows 更新，更新下。

## 问题描述
端口号被占用
## 解决
1. 打开控制台，执行 ` netstat -ano | findstr 1099` ，1099被占用的端口号。
2. 执行 `taskkill -pid 16704 -f` 16704 为占用次端口号的进程ID。

-----------