---
title: ASP.NET 知识收集与问题解决
tags: 'C#'
categories: 知识点
abbrlink: 51745
date: 2018-06-25 14:34:34
---

## 问题解决
### ASP.NET MVC部署报错
1. 部署时提示：**模块 DirectoryListingModule 通知 ExecuteRequestHandler 处理程序 StaticFile**
解决办法：在 **webconfig** 里 **system.webServer** 下的 **modules** 节点加一个属性为：**runAllManagedModulesForAllRequests="true"**
如果仍未解决，可查看.net版本是否过低（4.7的程序放在4.5上就不行）
2. 部署时提示：****配置错误****不能在此路径中使用此配置节。如果在父级别上锁定了该节，便会出现这种情况。锁定是默认设置的****(overrideModeDefault="Deny")****，或者是通过包含**** overrideMode="Deny" ****或旧有的****allowOverride="false" ****的位置标记明确设置的。****
解决办法：安装iis的时候还要安装 `asp.net`
-------