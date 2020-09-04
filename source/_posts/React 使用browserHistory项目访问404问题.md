---
title: React 使用browserHistory项目访问404问题
tags: 前端
categories: 前端
abbrlink: 45936
date: 2018-10-30 12:49:23
---
> 最近项目里面用到了React但是发布到iis站点之后,路由地址 刷新访问直接404错误。查阅资料之后发现是iis缺少配置URL重写 的问题导致的。下面我们来图形化配置，简单的配置下IIS

<!-- more -->
## IIS
安装[url-rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)
项目根目录新建`web.config`
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
<system.webServer>
  <rewrite>
    <rules>
      <rule name="React Routes" stopProcessing="true">
        <match url=".*" />
        <conditions logicalGrouping="MatchAll">
          <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
          <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          <add input="{REQUEST_URI}" pattern="^/(api)" negate="true" />
        </conditions>
        <action type="Rewrite" url="/" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
</configuration>
```
如果二级目录刷新后显示空白，手动把打包好的index.html中js引用路径 `./` 的 `.` 去掉。
## Nginx
```
server {
  server_name react.yahui.wang
  listen 80;

  root /wwwroot/ReactDemo/dist;
  index index.html;
  location / {
      try_files $uri /index.html;
    }
}
```

## Tomcat
找到conf目录下的web.xml文件，然后加上一句话让他定位回来
```
<error-page>
    <error-code>404</error-code>
    <location>/index.html</location>
</error-page>
```

## Apache
开启rewrite

```
sudo a2enmod rewrite
```

```
sudo vi /etc/apache2/sites-enabled/000-default.config
```
将其中的 AllowOverride None 修改为 AllowOverride All。

网站根目录新建`.htaccess`文件配置如下：

```
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteCond %{REQUEST_FILENAME} !-l
  RewriteRule . /index.html [L]
</IfModule>
```