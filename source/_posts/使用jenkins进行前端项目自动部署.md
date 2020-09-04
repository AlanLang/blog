---
title: 使用jenkins进行前端项目自动部署(ubuntu)
tags: 自动化
categories: 自动化
abbrlink: 27150
date: 2018-12-24 09:49:23
---
Jenkins 是一款业界流行的开源持续集成工具，广泛用于项目开发，具有自动化构建、测试和部署等功能。
<!-- more -->

## 安装
由于 jenkins是基于`java`环境运行的，所以首先需要安装`java`环境。
### 1.安装依赖包，使得add-apt-repository命令可以进行
```
apt-get install software-properties-common
```
### 2.通过add-apt-repository加载第三方的开源软件源
```
sudo add-apt-repository ppa:webupd8team/java
```
### 3.更新软件包列表，并安装jdk
```
sudo apt-get update
sudo apt-get install oracle-java8-installer
```
也可以安装 `openjdk `

```
sudo apt-get install openjdk-8-jdk
```
安装器会提示同意`oracle`的服务条款，选择`ok`,然后选择`yes` 即可。
### 4.通过查看java版本，来测试java环境是否安装成功
```
java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```
### 5.安装jenkins
将存储库密钥添加到系统。
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
```
我们将Debian包存储库地址附加到服务器的`sources.list`
```
echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
```
更新存储库
```
sudo apt-get update
```
安装`Jenkins`
```
sudo apt-get install jenkins
```
安装完成后，jenkins的文件目录如下所示
```
安装目录：/var/lib/jenkins  
日志目录：/var/log/jenkins/jenkins.log  
```
### 6.修改端口号
```
vi /etc/default/jenkins
```
### 7.启动
```
service jenkins start 
```
打开 `localhost:端口号`即可。
![](https://pic.xiaohuochai.site/blog/jenkins1.png)
然后在服务器的指定目录找到密码登录

## 安装配置`Apache2`
### 安装
```
sudo apt-get install apache2
```
### 配置多端口
以8080为例
```
vi /etc/apache2/ports.conf
```
添加监听`Listen 8080`
```
vi /etc/apache2/sites-available/000-default.conf
```
添加 `<VirtualHost *:8080>` 内容
例如:

```
<VirtualHost *:8080>
  ServerAdmin masterlink
  DocumentRoot /var/www/masterlink
  ErrorLog ${APACHE_LOG_DIR}/error_masterlink.log
  CustomLog ${APACHE_LOG_DIR}/access_masterlink.log combined
</VirtualHost>
```
### 启动
```
service apache2 start
```

### 问题解决
如果打开页面提示无权限，则将该页面的路径加入配置：
```
vi /etc/apache2/apache2.conf
```
加入以下内容

```
<Directory /home/alan/app>
  Options Indexes FollowSymLinks
  AllowOverride None
  Require all granted
</Directory>

```

## 设置GitHub Hook
### 配置GitHub
在工程主页面点击右上角的”Settings”，再点击左侧”Webhooks”，然后点击“Add webhook”，如下图： 
![](https://img-blog.csdn.net/20180120180649159?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
如下图，在”Payload URL”位置填入webhook地址，再点击底部的”Add webhook按钮”，这样就完成webhook配置了，今后当前工程有代码提交，GitHub就会向此webhook地址发请求，通知Jenkins构建：
![](https://img-blog.csdn.net/20180120181141062?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 生成Personal access tokens
Jenkins访问GitHub工程的时候，有的操作是需要授权的，所以我们要在GitHub上生成授权的token给Jenkins使用，这就是Personal access tokens，生成步骤如下： 

登录GitHub，进入”Settings”页面，点击左下角的”Developer settings”，如下图： 
![](https://img-blog.csdn.net/20180101100023785?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
跳转到”Developer settings”页面后，点击左下角的“Personal access tokens”，如下图
![](https://img-blog.csdn.net/20180101100149058?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可能会提示输入GitHub密码，输入后跳转到创建token的页面，如下图所示，输入title，再勾选”repo”和”admin:repo_hook”，再点击底部的”Generate token”按钮，就能产生一个新的access token，将此字符串复制下来，后面jenkins任务中会用到： 
![](https://img-blog.csdn.net/20180101100900159?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
### Jenkins配置
GitHub Plugin插件，在”系统管理->管理插件”位置检查此插件是否已经安装，没有的话请先安装；
配置GitHub，点击“系统管理->系统设置”，如下图： 
![](https://img-blog.csdn.net/20180121093316581?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
在系统设置页面找到”GitHub”，配置一个”GitHub Server”，如下图，”API URL”填写”https://api.github.com“，”Credentials”位置如下图红框所示，选择”Add->Jenkins”： 
![](https://img-blog.csdn.net/20180121094004625?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
弹出的页面中，”Kind”选择”Secret text”，”Secret”填入前面在GitHub上生成的Personal access tokens，Description随便写一些描述信息，如下图： 
![](https://img-blog.csdn.net/20180121094137737?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
填写完毕后，点击右侧的”Test connection”按钮，如果信息没有填错，显示的内容如下图所示： 
![](https://img-blog.csdn.net/20180121094800433?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
点击页面最底部的”保存”按钮；

### 新建项目时
#### 构建触发器
![](https://img-blog.csdn.net/20180121105320208?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#### 构建环境
![](https://img-blog.csdn.net/20180121105737723?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9saW5nX2NhdmFscnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 其他指令
### 免密码使用`sudo`直接使用`root`权限执行命令
vi /etc/sudoers

```
alan_ubuntu ALL=(ALL) NOPASSWD: /usr/sbin/servic
```