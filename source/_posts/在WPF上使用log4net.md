---
title: 在 WPF 上使用 log4net
tags: 'C#'
categories: 知识点
abbrlink: 26244
date: 2018-09-17 14:24:34
---
## 第一步
使用NuGet安装log4net
<!-- more -->

## 第二部
新建 log4net.config 并编辑
```
<?xml version="1.0"?>
<configuration>
  <configSections>
    <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net"/>
  </configSections>
  <log4net>
    <!--错误日志-->
    <appender name="RollingLogFileAppender" type="log4net.Appender.RollingFileAppender">
      <file value="log\\LogError\\"/>
      <appendToFile value="true"/>
      <rollingStyle value="Date"/>
      <datePattern value="yyyy\\yyyyMM\\yyyyMMdd'.txt'"/>
      <staticLogFileName value="false"/>
      <param name="MaxSizeRollBackups" value="100"/>
      <layout type="log4net.Layout.PatternLayout">
        <!--每条日志末尾的文字说明-->
        <!--输出格式-->
        <!--样例：2008-03-26 13:42:32,111 [10] INFO  Log4NetDemo.MainClass [(null)] - info-->
        <conversionPattern value="%newline %n记录时间：%date %n线程ID:[%thread] %n日志级别：  %-5level %n错误描述：%message%newline %n"/>
      </layout>
    </appender>
    <!--Info日志-->
    <appender name="InfoAppender" type="log4net.Appender.RollingFileAppender">
      <param name="File" value="Log\\LogInfo\\" />
      <param name="AppendToFile" value="true" />
      <param name="MaxFileSize" value="10240" />
      <param name="MaxSizeRollBackups" value="100" />
      <param name="StaticLogFileName" value="false" />
      <param name="DatePattern" value="yyyy\\yyyyMM\\yyyyMMdd'.txt'" />
      <param name="RollingStyle" value="Date" />
      <layout type="log4net.Layout.PatternLayout">
        <conversionPattern value="%newline %n记录时间：%date %n线程ID:[%thread] %n日志级别：  %-5level %n日志描述：%message%newline %n"/>
      </layout>
    </appender>

    <!--Error日志-->
    <logger name="logerror">
      <level value="ERROR" />
      <appender-ref ref="RollingLogFileAppender" />
    </logger>
    <!--Info日志-->
    <logger name="loginfo">
      <level value="INFO" />
      <appender-ref ref="InfoAppender" />
    </logger>
  </log4net>
</configuration>
```
## 第三步
将 log4net.config 设置为复制到生成目录，采用较新则复制的规则

## 第四步
在Properties -> Assemblyinfo.cs 里添加一行
```
[assembly: log4net.Config.XmlConfigurator(ConfigFile = "Log4Net.config", Watch = true)]
```
## 第五步
在app.xml.cs 里加入
```
protected override void OnStartup(StartupEventArgs e)
{
    log4net.Config.XmlConfigurator.Configure();
    base.OnStartup(e);
}
```