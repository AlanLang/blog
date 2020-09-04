---
title: 快速搭建ssr服务
tags: 文章
categories: 文章
abbrlink: 33668
date: 2019-12-05 18:48:23
---

## 使用ss-fly
### 安装git
```
yum -y install git
```
### 下载一键安装脚本
```
git clone -b master https://github.com/flyzy2005/ss-fly
// 或者：https://github.com/AlanLang/ss-fly.git
```

<!-- more -->

### 运行脚本
```
ss-fly/ss-fly.sh -i 密码 端口
```
### 其他操作
```
启动：/etc/init.d/ss-fly start
停止：/etc/init.d/ss-fly stop
重启：/etc/init.d/ss-fly restart
状态：/etc/init.d/ss-fly status
查看ss链接：ss-fly/ss-fly.sh -sslink
修改配置文件：vim /etc/shadowsocks.json
```
### 卸载
```
ss-fly/ss-fly.sh -uninstall
```
### 搭建ssr
```
ss-fly/ss-fly.sh -ssr
```
### 相关操作ssr命令
```
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status
配置文件路径：/etc/shadowsocks.json
日志文件路径：/var/log/shadowsocks.log
代码安装目录：/usr/local/shadowsocks
```
### 卸载ssr
```
 ./shadowsocksR.sh uninstall
```
## 一键开启BBR加速
```
ss-fly/ss-fly.sh -bbr
```
装完后需要重启系统，输入y即可立即重启，或者之后输入reboot命令重启。
判断BBR加速有没有开启成功。输入以下命令：
```
sysctl net.ipv4.tcp_available_congestion_control
```
如果返回值为：
```
net.ipv4.tcp_available_congestion_control = bbr cubic reno
```

## 使用shadowsocks-all
### 下载并安装
```
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```
### 安装bbr加速
```
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```
选2 安装BBR plus
重启
```
./tcp.sh
```
选 7 使用BBR plus 加速