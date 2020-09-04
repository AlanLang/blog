---
title: CentOS 7 安装 GitLab
tags: Linux
categories: Linux
abbrlink: 30939
date: 2018-03-16 12:49:23
---
[教程](https://www.cnblogs.com/wenwei-blog/p/5861450.html) | [汉化](https://www.cnblogs.com/cheng95/p/8037865.html) | [参考](http://www.souvc.com/?p=2733#gitlab)

#### 1.安装gitlab所需要的依赖
```
sudo yum install curl policycoreutils openssh-server openssh-clients
```
<!-- more -->
##### 2.使sshd服务自动启动 
```
sudo systemctl enable sshd
```
#### 3.启动sshd服务
```
 sudo systemctl start sshd
```
#### ４.安装邮件服务器
```
sudo yum install postfix
```
#### 5.使邮件服务器postfix自启动 
```
sudo systemctl enable postfix
```
#### 6. 启动邮件服务器postfix 
```
sudo systemctl start postfix
```
#### 7.添加GitLab仓库,并安装到服务器上
```
curl  -sS http://packages.gitlab.cc/install/gitlab-ce/script.rpm.sh | sudo bash
```
#### 8.安装gitlab
```
sudo yum install gitlab-ce
```
#### 9.启动，配置
```
sudo gitlab-ctl reconfigure
```
#### 10.修改gitlab配置文件指定服务器ip和自定义端口
```
vi  /etc/gitlab/gitlab.rb
```
```
pages_external_url ""
unicorn['listen'] = '127.0.0.1'
unicorn['port'] = 9000
```
在防火墙开放指定端口

#### 11.重新配置
```
sudo gitlab-ctl reconfigure
```
#### 12.重启
```
gitlab-ctl restart
```
### 配置发送邮件
[参考地址](https://docs.gitlab.com/omnibus/settings/smtp.html#amazon-ses)
QQMail
```
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "xxxx@xx.com"
gitlab_rails['smtp_password'] = "password"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = 'xxxx@xx.com'
```
更新配置
```
sudo gitlab-ctl reconfigure
```
备份
```
sudo gitlab-rake gitlab:backup:create 
```
还原备份
```
sudo cp 140623891_gitlab_backup.tar  /var/opt/gitlab/backups/   
sudo gitlab-ctl stop unicorn  
sudo gitlab-ctl stop sidekiq  
sudo gitlab-rake gitlab:backup:restore BACKUP=140623891   -- 备份文件名的时间戳前缀  
sudo gitlab-ctl start  
sudo gitlab-rake gitlab:check SANITIZE=true  
```
## 定时备份
执行`crontab -e`命令
```
crontab -e
```

输入以下内容，设置每天凌晨2:00定时自动备份
```
0 2 * * * gitlab-rake gitlab:backup:create
```

## Centos 7 防火墙操作
查看已经开放的端口：
```
firewall-cmd --list-ports
```
开启端口
```
firewall-cmd --zone=public --add-port=80/tcp --permanent
```
重启防火墙
```
firewall-cmd --reload #重启firewall
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```
