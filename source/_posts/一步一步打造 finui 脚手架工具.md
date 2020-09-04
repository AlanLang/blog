---
title: 一步一步打造 finui 脚手架工具
tags: 前端
categories: 前端
abbrlink: 15520
date: 2019-06-17 18:48:23
---
## 工程结构
工程基于nodejs 8.4以及ES6进行开发，目录结构如下

```
/bin  # ------ 命令执行文件
/lib  # ------ 工具模块
package.json
```
<!-- more -->

## 使用commander.js开发命令行工具
nodejs内置了对命令行操作的支持，node工程下`package.json`中的`bin`字段可以定义命令名和关联的执行文件。

``` json
{
  "name": "finui-cli",
  "version": "1.0.0",
  "description": "finui脚手架工具",
  "bin": {
    "bi": "./bin/command.js"
  },
}
```

经过这样配置的nodejs项目，在使用`-g`选项进行全局安装的时候，会自动在系统的`[prefix]/bin`目录下创建相应的符号链接（symlink）关联到执行文件。如果是本地安装，这个符号链接会生成在`./node_modules/.bin`目录下。这样做的好处是可以直接在终端中像执行命令一样执行nodejs文件。关于`prefix`，可以通过`npm config get prefix`获取。

### hello,commander.js
在bin目录下创建一个`command.js`文件，用于处理命令行的逻辑。

```
touch ./bin/command.js
```
[commander.js](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fcommander.js)可以自动的解析命令和参数，合并多选项，处理短参，等等，功能强大，上手简单。具体的使用方法可以参见官方文档。
在`command.js`编写命令行的入口逻辑

``` js
const program = require('commander')  // npm i commander -D

program.version('1.0.0')
	.usage('<command> [项目名称]')
	.command('hello', 'hello')
	.parse(process.argv)
```
然后继续在`bin`目录下新建文件`command-hello.js`，并写一个打印语句：

```
console.log('hello, commander')
```
这样，通过node命令测试一下

```
node ./bin/command hello
```
不出意外，可以在终端上看到一句话：`hello, commander`。

## 创建项目
### 新建项目目录
在正式下载项目模板之前，还需要判断当前目录是否有相同名称的项目，如果没有则开始创建项目。
在`lib`目录下创建`createProject.js`

```js
const path = require('path')
const fs = require('fs')
const glob = require('glob')
module.exports = function (projectName){
  return new Promise(function(resolve, reject){
    const list = glob.sync('*')
    let rootName = path.basename(process.cwd())
    if (list.length){
      if (list.filter(name => {
        const fileName = path.resolve(process.cwd(), path.join('.', name))
        const isDir = fs.statSync(fileName).isDirectory()
        return name.indexOf(projectName) !== -1 && isDir
      }).length !== 0) {
        reject(`${projectName} directory is exist`)
      }else{
        resolve(Promise.resolve(projectName))
      }
    } else if (rootName === projectName){
      inquirer.prompt([
        {
          name: 'buildInCurrent',
          message: 'The current directory is empty and the directory name is the same as the project name. Do you want to create a new project directly in the current directory?',
          type: 'confirm',
          default: true
        }
      ]).then(answer => {
        resolve(Promise.resolve(answer.buildInCurrent ? projectName : '.'))
      })

    }else {
      resolve(Promise.resolve(projectName))
    }
  })
}
```

### 使用download-git-repo下载项目模板
在`lib`目录下创建`download.js`

```js
/**
 * A library of download-git-repo to download according the git address
 * Alan<alan@fanruan.com>
 */
const download = require('download-git-repo')
const ora = require('ora') // show download spinner

module.exports = function (url, target) {
  return new Promise(function(resolve, reject){
    const spinner = ora(`it is downloading template, source address: https://github.com/${url}`)
    spinner.start()
    download('direct:https://github.com/' + url, target, { clone: true }, (err) => {
      if (err) {
        spinner.fail()
        reject(err)
      } else {
        spinner.succeed()
        resolve(target)
      }
    })
  })
}
```
`download-git-repo` 的作用就是从GitHub等托管代码的地方将模板代码下载下来。

## 对模板进行自定义处理
通常我们可能会希望项目模板中有些文件或者代码可以动态处理。比如：
新项目的名称、版本号、描述等信息等，可以通过脚手架的交互进行输入，然后将输入插入到模板中
对于这类情况，我们还需要借助其他工具包来完成。
### 使用inquirer.js处理命令行交互
对于命令行交互的功能，可以用[inquirer.js](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FSBoudrias%2FInquirer.js)来处理。用法其实很简单：

```js
const inquirer = require('inquirer')  // npm i inquirer -D

inquirer.prompt([
  {
    name: 'projectName',
    message: '请输入项目名称'
  }
]).then(answers => {
  console.log(`你输入的项目名称是：${answers.projectName}`)
})
```
`prompt()`接受一个问题对象的数据，在用户与终端交互过程中，将用户的输入存放在一个答案对象中，然后返回一个Promise，通过`then()`获取到这个答案对象。so easy！

在`lib`目录下创建`changeProject.js`, 通过命令行交互获取项目名称、版本号、描述等信息，并替换到`package.json`中。

```js
const inquirer = require('inquirer')
const fs = require('fs')
module.exports = async function (projectName){
  const fileName = `./${projectName}/package.json`
  const package = await fs.readFileSync(fileName)
  const packageJson = JSON.parse(package.toString());
  const answers = await getProjectMessage(projectName);
  packageJson.name = answers.projectName
  packageJson.version = answers.projectVersion
  packageJson.description = answers.projectDescription
  await fs.writeFileSync(fileName, JSON.stringify(packageJson, null, 4))
}

async function getProjectMessage(projectName){
  return inquirer.prompt([
    {
      name: 'projectName',
      message: 'name',
      default: projectName
    },
    {
      name: 'projectVersion',
      message: 'version',
      default: '1.0.0'
    },
    {
      name: 'projectDescription',
      message: 'description',
      default: `A project named ${projectName}`
    }
  ])
}
```
就这样，一个基本的脚手架工具就已经完成了。
## 上传到npm
NPM是随同NodeJS一起安装的javascript包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：

1. 允许用户从NPM服务器下载别人编写的第三方包到本地使用。
1. 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
1. 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。


#### 1. 注册一个npm账号
前往[NPM](https://www.npmjs.com/)官网进行注册

#### 2. 登录npm
使用终端命令行，如果是第一次发布包，执行以下命令，然后输入前面注册好的NPM账号，密码和邮箱，将提示创建成功。

```
npm adduser
```
如果不是第一次发布包，执行以下命令进行登录，同样输入NPM账号，密码和邮箱

```
npm login
```
**注意：npm adduser成功的时候默认你已经登陆了，所以不需要再进行npm login了**

接着先进入项目文件夹下，然后输入以下命令进行发布

```
npm publish
```
本教程已发布到[npm](https://www.npmjs.com/package/finui-cli)上，可以直接安装体验：

```
npm i -g finui-cli
```