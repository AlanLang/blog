---
title: Angular CLI 命令行收集
tags: 前端
categories: 前端
abbrlink: 27930
date: 2018-07-09 08:48:23
---

### 安装 Angular CLI
```
npm install -g @angular/cli
```
<!-- more -->
### 创建新应用
```
ng new angular-tour-of-heroes
```

### 启动应用服务器
```
ng serve --open
```

### 创建新组建
```
ng generate component heroes
```
或者
```
ng g c heroes
```

### 创建服务
```
ng generate service hero
```

### 创建类
```
ng generate class hero
```

## 创建特性模块
```
ng generate module CustomerDashboard
```

### 添加 AppRoutingModule 路由器
```
ng generate module app-routing --flat --module=app
```

### 添加模拟数据服务器模块
```
npm install angular-in-memory-web-api --save
```

## 添加 Ant Desinger of Angular
```
ng add ng-zorro-antd
```
## 问题解决
### build 后引用路径错误
解决方案：
在文件 `package.json` 文件的 `script` 
```
"build":"ng build --base-href ./"
```
### 使用 `ngx-echart` 报错
解决方案：
```
npm install --save rxjs-compat
```
### 使用`formGroup`报错
解决方案：
```
import { FormsModule,ReactiveFormsModule } from '@angular/forms';
...
imports: [
	// 其他引用
	FormsModule,
	ReactiveFormsModule
],
```