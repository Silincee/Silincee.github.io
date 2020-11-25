---
layout: post
title: "SpringCloud Bus消息总线"
date: 2020-11-25 20:11:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# 概述

## 主要功能

Spring Cloud Bus配合Spring Cloud Config使用可以实现配置分布式自动刷新功能。**Bus支持两种消息代理：RabbitMQ和Kafka**。（⚠️通知的是客户端）

![image-20201125203859434](/assets/imgs/image-20201125203859434.png)

Spring Cloud Bus能管理和传播分布式系统间的消息，就像一个分布式执行器， 可用于广播状态更改、事件推送等，也可以当作微服务间的通信通道。（⚠️通知的是中心）

该方式更加适合，图一不适合的理由理由如下：

- 打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新职责
- 破坏了微服务各节点的对等性
- 有一定的局限性。例如，微服务在迁移时，它的网络地址常常会发生变化，此时如果想要做到自动刷新，那就会增加更多的修改

![image-20201125204301676](/assets/imgs/image-20201125204301676.png)



## 为什么被称为总线

***什么是总线：***

在微服务架构的系统中，通常会使用**轻量级的消息代理**来构建一个**共用的消息主题**， 并让系统中所有微服务实例都连接上来。由于**该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线**。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题 上的实例都知道的消息。

***基本原理：***

ConfigClient实例都监听MQ中同一个topic（默认是springCloudBus）。当一个服务刷新数据的时候，它会把这个信息放入到Topic中，这样其它监听同一Topic的服务就能得到通知，然后去更新自身的配置。



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
cd /Users/silince/Applications/rabbitmq/rabbitmq_server-3.7.14
sbin/rabbitmq-server
# RabbitMQ 启动插件
cd /Users/lidong/javaEE/rabbitmq_server-3.6.6/sbin
sudo ./rabbitmq-plugins enable rabbitmq_management（执行一次以后不用再次执行）
# 登陆管理界面 账号密码初始默认都为guest
http://localhost:15672/
```





# SpringCloud Bus动态刷新全局广播

1.演示广播效果，增加复杂度，再以3355为模板再制作一个3366

- 必须先具备良好的RabbitMQ环境先，演示广播效果，增加复杂度，再以3355为模板再制作一个3366

- 新建cloud-config-client-3366

2.设计思想

1) 利用消息总线触发一个客户端/bus/refresh,而刷新所有客户端的配置

2) 利用消息总线触发一个服务端ConfigServer的/bus/refresh端点,而刷新所有客户端的配置（⭐️更加推荐）

![image-20201125221609657](/assets/imgs/image-20201125221609657.png)

3.给cloud-config-center-3344配置中心**服务端**添加消息总线支持

- POM

```xml
<!--添加消息总线RabbitMQ支持-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

- YML

```yml
# rabbitmq相关配置
rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    
# rabbitmq相关配置,暴露bus刷新配置的端点
management:
  endpoints: # 暴露bus刷新配置的端点
    web:
      exposure:
        include: 'bus-refresh'
```

4.给cloud-config-center-3355**客户端**添加消息总线支持

- POM 同上
- YML

```yml
#rabbitmq相关配置 15672是Web管理界面的端口；5672是MQ访问的端口
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

5.给cloud-config-center-3366**客户端**添加消息总线支持

- POM 和 YML 同上

6.测试

- 修改git配置文件的版本号，发送post请求`curl -X POST "http://localhost:3344/actuator/bus-refresh"`
- 配置中心:http://config-3344.com:3344/main/config-dev.yml
- 客户端
  - http://localhost:3355/configInfo
  - http://localhost:3366/configInfo
- ***一次修改，广播通知，处处生效!!!***



***RabbitMQ情况：***

![WX20201125-223026](/assets/imgs/WX20201125-223026.png)



# SpringCloud Bus动态刷新定点通知

不想全部通知，只想定点通知。即指定具体某一个实例生效而不是全部。

公式：http://localhost:配置中心的端口号/actuator/bus-refresh/{destination}

- 其中`destination`目的地等于`微服务名称:端口号`

`/bus/refresh`请求不再发送到具体的服务实例上，而是发给`config server`并通过`destination`参数类指定需要更新配置的服务或实例

案例：我们这里以刷新运行在3355端口上的config-client为例，达到只通知3355不通知3366的效果。

- `curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"`



# 通知总结

![image-20201125223832779](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201125223832779.png)