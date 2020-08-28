---
layout: post
title:  " Docker安装redis"
date:   2019-12-19 12:20:06 +0800--
categories: [服务器, 数据库]
tags: [docker,redis ]  
---
# Docker安装redis

##  下载redis镜像

```undefined
docker pull redis:5.0.7
```



## 配置data、conf

```kotlin
cd /root
mkdir -p ./docker/redis/data
mkdir -p ./docker/redis/conf
```



## 配置redis.conf

拷贝原配置redis.conf文件到./docker/redis/conf，并修改以下内容

```shell
bind 127.0.0.1 #注释掉这部分，用来限制redis只能本地访问
protected-mode no #默认yes表示开启保护模式，用来限制redis只能本地访问
daemonize no #默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程（可选）；改为yes会使配置文件方式启动redis失败
dir ./ #输入本地redis数据库存放文件夹（可选）
appendonly yes #redis持久化（可选）
dir /data # ⚠️ 忘记改这个可恶心死我了 指定本地数据库路径 
```

ps:

```css
protected-mode 是在没有显示定义 bind 地址（即监听全网断），又没有设置密码 requirepass
时，只允许本地回环 127.0.0.1 访问。 也就是说当开启了 protected-mode 时，如果你既没有显示的定义了 bind
监听的地址，同时又没有设置 auth 密码。那你只能通过 127.0.0.1 来访问 redis 服务
```

## 启动redis

### 创建并运行一个名为 myredis 的容器

```shell
docker run -p 6379:6379 -v /root/docker/redis/data:/data -v /root/docker/redis/conf/redis.conf:/etc/redis/redis.conf --privileged=true --name myredis -d redis:5.0.7 redis-server /etc/redis/redis.conf
```

### 命令分解

```bash
docker run \
-p 6379:6379 \ # 端口映射 宿主机:容器
-v /docker/redis/data:/data: \ # 映射数据目录
-v /docker/redis/conf/redis.conf:/etc/redis/redis.conf \ # 挂载配置文件 
--privileged=true \ # 给与一些权限
--name myredis \ # 给容器起个名字
-d docker.io/redis:5.0.7 redis-server /etc/redis/redis.conf # deamon 运行 服务使用指定的配置文件
```

## 查看状态

```undefined
docker ps
```



## 客户端测试

```bash
docker exec -it myredis env LANG=C.UTF-8 /bin/bash
redis-cli -a yourpassword
```



