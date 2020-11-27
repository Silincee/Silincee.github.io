---
layout: post
title: "mac rabbitMQ环境配置"
date: 2020-11-25 19:11:16 +0800--
categories: [Java, 分布式]
tags: [Java, 消息队列, rabbitMQ]  

---

# RabbitMQ环境配置

1.安装Erlang，[下载地址](http://www.erlang.org/download/otp_src_R16B03.tar.gz)

```shell
tar -zxvf otp_src_R16B03.tar.gz
cd otp_src_R16B03
./configure   
make
sudo make install
# 环境测试
erl
```

2.安装RabbitMQ，[下载地址](https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.14/)

```shell
tar -zxvf rabbitmq-server-mac-standalone-3.7.14.tar.xz
# 启动服务
cd /Users/silince/Applications/rabbitmq/rabbitmq_server-3.7.14
./sbin/rabbitmq-server
# 关闭服务
./rabbitmqctl stop
# RabbitMQ 启动插件
cd /Users/lidong/javaEE/rabbitmq_server-3.6.6/sbin
sudo ./rabbitmq-plugins enable rabbitmq_management（执行一次以后不用再次执行）
# RabbitMQ 关闭插件
sudo ./rabbitmq-plugins disable rabbitmq_management
# 登陆管理界面 账号密码初始默认都为guest
http://localhost:15672/
```

