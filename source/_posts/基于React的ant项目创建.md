---
title: 基于React的ant项目创建
tags: 前端
categories: 前端
abbrlink: 25352
date: 2018-05-17 12:49:23
---

## 安装和初始化
安装 create-react-app 工具 (已安装则跳过)

```
npm install -g create-react-app
```
<!-- more -->
然后新建一个项目。

```
create-react-app antd-demo
```

安装并引入 antd。
```
cd antd-demo
npm add antd
```

如果是 antd mobile，则引入
```
npm install antd-mobile --save
```

## 配置按需加载ant
引入 react-app-rewired 并修改 package.json 里的启动配置。

```
npm add react-app-rewired --dev
```

```
/* package.json */
"scripts": {
-   "start": "react-scripts start",
+   "start": "react-app-rewired start",
-   "build": "react-scripts build",
+   "build": "react-app-rewired build",
-   "test": "react-scripts test --env=jsdom",
+   "test": "react-app-rewired test --env=jsdom",
}
```

然后在项目根目录创建一个 `config-overrides.js` 用于修改默认配置。
```
module.exports = function override(config, env) {
  // do stuff with the webpack config...
  return config;
};
```

**使用 babel-plugin-import**

babel-plugin-import 是一个用于按需加载组件代码和样式的 babel 插件，现在我们尝试安装它并修改 `config-overrides.js` 文件。

```
npm add babel-plugin-import --dev
```

```
+ const { injectBabelPlugin } = require('react-app-rewired');
  module.exports = function override(config, env) {
+   config = injectBabelPlugin(['import', { libraryName: 'antd', libraryDirectory: 'es', style: 'css' }], config);
    return config;
  };
```

然后直接引用and的组件即可,例如：
```
import { Button } from 'antd';
```