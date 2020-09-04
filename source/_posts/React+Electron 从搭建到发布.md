---
title: React+Electron 从搭建到发布
tags: 前端
categories: 前端
abbrlink: 12273
date: 2019-01-09 09:28:23
---

## 准备工作
安装`node`环境
https://nodejs.org/zh-cn/
安装过程不做赘述。

安装`react`脚手架`create-react-app`和[yarn](https://github.com/yarnpkg/yarn/)
```
npm install -g create-react-app yarn
```

创建`React`项目
脚手架安装完成后，执行以下命令，创建一个名为`react-electron-demo`的应用
```
create-react-app react-electron-demo
```
<!-- more -->

## 引入`Electron`
安装 `electron`
```
cd react-electron-demo
yarn add electron --dev
yarn add electron-is-dev
```
根目录新建入口文件`main.js`
```
const electron = require('electron');
const app = electron.app;
const BrowserWindow = electron.BrowserWindow;

const path = require('path');
const url = require('url');
const isDev = require('electron-is-dev');

let mainWindow;

function createWindow() {
  mainWindow = new BrowserWindow({width: 900, height: 680});
  mainWindow.loadURL(isDev ? 'http://localhost:3000' : `file://${path.join(__dirname, './build/index.html')}`);
  mainWindow.on('closed', () => mainWindow = null);
}

app.on('ready', createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', () => {
  if (mainWindow === null) {
    createWindow();
  }
});
```
将入口文件加入到package.json中
```
"main": "main.js",
"homepage": ".",
```
添加 `npm scripts`
```
"electron": "electron .",
```
启动
```
yarn start
// 新建一个终端
yarn electron
```
启动后效果如图：
![](http://dada-image-bed.oss-cn-shenzhen.aliyuncs.com/19-1-9/18305917.jpg)

## 优化
### 使用`concurrently`并行运行
同时开两个终端有点繁琐，所以可以借助工具`concurrently`。
安装`concurrently`
```
yarn add concurrently --dev
```
添加 `npm scripts`
```
"dev": "concurrently \"yarn start\" \"electron .\""
```

### 禁止启动时在浏览器中打开
根目录新建文件`.env`，输入：
```
BROWSER=none
```
保存后重新启动即可

### 优化启动顺序
由于electron启动需要先等react启动完毕，所以可以使用工具`wait-on`。
安装`wait-on`
```
yarn add wait-on --dev
```
修改`npm scripts`
```
"dev": "concurrently \"yarn start\" \"wait-on http://localhost:3000 && electron .\""
```

## 打包发布
安装`electron-builder`
```
yarn add electron-builder --dev
```
在`package.json`中添加`build`字段
```
"build": {
  "appId": "com.example.electron-cra",
  "files": [
    "build/**/*",
    "node_modules/**/*",
    "public/**/*",
    "main.js"
  ],
  "directories":{
    "buildResources": "assets"
  }
}
```
添加 `npm scripts`
以windows平台为例，其他平台请参考`electron-builder`文档
```
"package": "yarn build && electron-builder -c.extraMetadata.main=main.js --win --x64"
```
打包
```
yarn package
```
打包后的文件会在`dist`目录中

![](http://dada-image-bed.oss-cn-shenzhen.aliyuncs.com/19-1-9/95561492.jpg)

此教程的源码已托管在github：
https://github.com/AlanLang/react-electron-demo

谢谢您的阅读，如发现本文有任何不妥，欢迎指正，不胜感激。