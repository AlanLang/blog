---
title: 使用Docker部署Drone
tags: docker
categories: 教程
date: 2020-10-21 13:53:23
---

## 新建 GitHub 应用
登录 GitHub，在 https://github.com/settings/applications/new 新建一个应用。
![](https://alan-picpack.oss-cn-hangzhou.aliyuncs.com/github_application_create.png)
接下来查看这个应用的详情，记录 Client ID 和 Client Secret，之后配置 Drone 会用到。

<!-- more -->

## 配置 Drone
我们通过使用`Docker Compose`来启动 Drone，编写`docker-compose.yml`文件。
```yml
version: "3"
services:
  drone-server:
    image: drone/drone
    ports:
      - "8005:80"
      - "7003:443"
    container_name: my_drone
    restart: always
    volumes:
      - "/volume1/docker/drone:/data"
    environment:
      - DRONE_AGENTS_ENABLED=true
      - DRONE_SERVER_HOST=${DRONE_SERVER_HOST:-https://drone.yeasy.com}
      - DRONE_SERVER_PROTO=${DRONE_SERVER_PROTO:-https}
      - DRONE_RPC_SECRET=${DRONE_RPC_SECRET:-secret}
      - DRONE_GITHUB_SERVER=https://github.com
      - DRONE_GITHUB_CLIENT_ID=${DRONE_GITHUB_CLIENT_ID}
      - DRONE_GITHUB_CLIENT_SECRET=${DRONE_GITHUB_CLIENT_SECRET}
	    - DRONE_USER_CREATE=username:alanlang,admin:true

  drone-agent:
    image: drone/drone-runner-docker:1
    restart: always
    ports:
      - "3000:3000"
    depends_on:
      - drone-server
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    environment:
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_HOST=drone-server
      - DRONE_RPC_SECRET=${DRONE_RPC_SECRET:-secret}
      - DRONE_RUNNER_NAME=${HOSTNAME:-demo}
      - DRONE_RUNNER_CAPACITY=2
      - DOCKER_API_VERSION=1.39
    dns: 114.114.114.114
```

新建 .env 文件，输入变量及其值

```
# 必填 服务器地址，例如 drone.domain.com
DRONE_SERVER_HOST=
DRONE_SERVER_PROTO=https
DRONE_RPC_SECRET=secret
HOSTNAME=demo
# 必填 在 GitHub 应用页面查看
DRONE_GITHUB_CLIENT_ID=
# 必填 在 GitHub 应用页面查看
DRONE_GITHUB_CLIENT_SECRET=
```

## 启动 Drone

```
docker-compose up -d
```

## 使用Drone本地部署docker应用
编写`.drone.yml`

```yml
kind: pipeline
name: cicd

steps:
  - name: test
    image: docker/compose
    volumes:
      - name: docker_socket
        path: /var/run/docker.sock
    commands:
      - docker-compose up -d

volumes:
  - name: docker_socket
    host:
      path: /var/run/docker.sock
```

## 问题解决
### 无法使用docker
```
Couldn't connect to Docker daemon at http+docker://localhost - is it running?
If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.
```
解决
需要挂载docker环境 https://discourse.drone.io/t/solved-run-docker-compose-within-drone-yml/6563

### 无法使用本地挂载
```
cicd: linter: untrusted repositories cannot mount host volumes
```
解决
手动开启项目的Trusted
![](https://img-blog.csdnimg.cn/20200618013525763.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDI1MDcw,size_16,color_FFFFFF,t_70)