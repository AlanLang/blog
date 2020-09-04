---
title: Linux 笔记
tags: Linux
categories: Linux
abbrlink: 13448
date: 2019-04-19 09:49:23
---
## 环境变量
### 方式1 修改`profile `
```
vi /etc/profile
```
在文档末尾添加

```
export PATH=$PATH:[路径]
```
配置生效

```
source /etc/profile
```

<!-- more -->
### 方式2 建立软链接
例如配置node的环境变量，node在`/usr/local/src/nodejs/bin`中：

```
ln -s /usr/local/src/nodejs/bin/node  /usr/local/bin/node
```
## crontab
## 配置定时任务

```
crontab -e
```
### 服务操作命令

```
service crond start //启动服务  
service crond stop //关闭服务  
service crond restart //重启服务  
service crond reload //重新载入配置
```
### 配置日志

```
vim /etc/rsyslog.d/50-default.conf
service rsyslog restart
```
### 查看日志

```
more /var/log/cron.log
```
### 注意事项
* 环境变量问题
* 权限问题
* 定时执行的sh脚本的环境变量问题

## Docker
### 常用命令
* `docker exec -it centos1 bash` 进入指定镜像
* `docker ps -all` 展示镜像
* `docker commit b5926410fe60 herong/centos7-ssh` 将容器保存为镜像 b5926410fe60 为容器id herong/centos7-ssh为镜像名
* `docker run -p 9003:22 -p 9100-9150:9100-9150 --name="ububtu-alan" -d alan/ubuntu /usr/sbin/sshd -D` 运行

### 新建镜像
`vi Dockerfile`

```Dockerfile
#制定node镜像的版本
FROM node:10.15.3
#声明作者
MAINTAINER alan
#移动当前目录下面的文件到app目录下
ADD . /app/
#进入到app目录下面，类似cd
WORKDIR /app
#安装依赖
RUN npm i yarn
RUN yarn
#对外暴露的端口
EXPOSE 9100
#程序启动脚本
CMD ["yarn", "docker"]
```
`docker build -t [文件夹名称] .`

## pm2
### 安装
```
npm install pm2 -g
```
### 启动
```
pm2 start npm --name project01 -- start
```
### 常用命令
```
pm2 list               # 显示所有进程状态
pm2 monit              # 监视所有进程
pm2 logs               # 显示所有进程日志
pm2 stop all           # 停止所有进程
pm2 restart all        # 重启所有进程
pm2 reload all         # 0 秒停机重载进程 (用于 NETWORKED 进程)
pm2 stop 0             # 停止指定的进程
pm2 restart 0          # 重启指定的进程
pm2 startup            # 产生 init 脚本 保持进程活着
pm2 web                # 运行健壮的 computer API endpoint (http://localhost:9615)
pm2 delete 0           # 杀死指定的进程
pm2 delete all         # 杀死全部进程
pm2 startup            # 开机启动
pm2 save               # 保存当前应用列表
```

## 后台执行程序
例如执行pso
```
nohup  ./pso > pso.log 2>&1 &
```

```
# 查看任务，返回任务编号n和进程号
jobs -l

# 挂起当前任务
ctrl+z

# 将编号为n的任务转后台运行
bg %n

# 将编号为n的任务转前台运行
# fg %n

# 结束当前任务
ctrl+c 

# 设置程序父进程为1，不中断
setsid ./test.sh &

# 查看指定任务详细
ps -ef | grep test

# 显示当窗口父进程ID
echo $$
```