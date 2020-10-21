---
title: DockerCompose使用教程
tags: docker
categories: 教程
abbrlink: 60376
date: 2020-09-01 13:53:23
---

## 安装
参考官网:https://docs.docker.com/compose/install/
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## 使用
以构建jenkins为例
新建 Dockerfile
```
vi Dockerfile
```
```
FROM jenkins/jenkins:lts

USER root

RUN apt-get update
RUN apt-get install -y libltdl-dev
```

新建 docker-compose.yml
```
vi docker-compose.yml
```
```
version: "3"
services:
 jenkins:
  build: .
  image: my_jenkins
  ports:
    - "8090:8080"
    - "50000:50000"
  container_name: my_jenkins
  volumes:
    - "/home/jenkins_home:/var/jenkins_home"
    - "/var/run/docker.sock:/var/run/docker.sock"
    - "/usr/bin/docker:/usr/bin/docker"
```
执行
```
docker-compose up -d
```