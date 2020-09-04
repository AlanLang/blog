---
title: 在 TypeScript 中使用 ESLint
tags: 前端
categories: 前端
abbrlink: 30810
date: 2019-04-14 12:00:34
---
### 使用eslint而不使用tslint
由于性能问题，TypeScript 官方决定全面采用 ESLint，甚至把仓库（Repository）作为测试平台，而 ESLint 的 TypeScript 解析器也成为独立项目，专注解决双方兼容性问题。

JavaScript 代码检验工具 ESLint 在 TypeScript 团队发布全面采用 ESLint 之后，发布 typescript-eslint 项目，以集中解决 TypeScript 和 ESLint 兼容性问题。而 ESLint 团队将不再维护 typescript-eslint-parser，也不会在 Npm 上发布，TypeScript 解析器转移至Github 的 typescript-eslint/parser。
<!-- more -->
### 安装
```
yarn add --dev eslint
yarn add --dev @typescript-eslint/eslint-plugin
yarn add --dev @typescript-eslint/parser
```
### 配置 eslint
新建文件`.eslintrc.js`

```
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  extends: ['plugin:@typescript-eslint/recommended'],
}
```

### 配置 VScode
下载`eslint`插件，设置保存修改时自动修复：

```
"eslint.validate": [
  "javascript",
  "javascriptreact",
  "typescriptreact",
  {
    "language": "typescript",
    "autoFix": true
  }
],
"eslint.enable": true,
"eslint.autoFixOnSave": true
```
### 常用规则
控制缩进为两个空格

```
"@typescript-eslint/indent": ["error", 2]
```
字符串总为一个单引号包裹

```
"quotes": [1, "single"]
```
只要求自定义的方法设置返回类型

```
"@typescript-eslint/explicit-function-return-type": ["warn", {
  allowExpressions: true
}]
```
关闭`any`类型时的警告

```
"@typescript-eslint/no-explicit-any": ["off"]
```

