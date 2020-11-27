---
layout: post
title: "SpringCloud Stream消息驱动"
date: 2020-11-26 23:11:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# 消息驱动概述

> 屏蔽底层消息中间件的差异，降低切换版本，统一消息的编程模型。
>
> 官网：https://spring.io/projects/spring-cloud-stream#overview
>
> 官网API：https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.1.RELEASE/reference/html
>
> Spring Cloud Stream中文指导手册：https://m.wang1314.com/doc/webapp/topic/20971999.html

## 什么是SpringCloudStream

官方定义Spring Cloud Stream是一个沟建消息驱动微服务的框架。

应用程序通过inputs或者outputs来与Spring Cloud Stream中**binder对象**交互。

**通过我们配置来binding（绑定），而Spring Cloud Stream的binder对象负责与消息中间件交互。**

所以，我们只需要搞清楚如何与Spring Cloud Stream交互就可以方便使用消息驱动的方式。

通过使用Spring Integration来连接消息代理中间件以实现消息事件驱动。
Spring Cloud Stream为一些供应商的消息中间件产品提供了个性化的自动化配置实现，引用了发布订阅、消费组、分区的三个核心概念。

***目前仅支持RabbitMQ、Kafka.***

## 设计思想

### 标准MQ

- 生产者/消费者之间靠**消息**媒介传递信息内容 

- 消息必须走特定的**通道**(消息通道MessageChannel)

- 消息通道里的消息如何被消费呢，谁负责收发**处理**

  消息通道`MessageChannel`的子接口`SubscribableChannel`,由`MessageHandler`消息处理器订阅

![image-20201127110714707](/assets/imgs/image-20201127110714707-6446772.png)

### 为什么用Cloud Stream

比方说我们用到了RabbitMQ和Kafka，由于这两个消息中间件的架构上的不同，像RabbitMQ有exchange， kafka有 Topic和Partitions分区。

这些中间件的差异性导致我们实际项目开发给我们造成了一定的困扰，我们如果用了两个消息队列的其中一种，后面的业务需求，我想往另外一种消息队列进行迁移，这时候无疑就是一个灾难性的， **一大堆东西都要重新推倒重新做**，因为它跟我们的系统耦合了，这时候springcloud Stream给我们提供了一种解耦合的方式。

![image-20201127110959509](/assets/imgs/image-20201127110959509-6446740.png)

### stream凭什么可以统一底层差异

在没有绑定器这个概念的情况下，我们的SpringBoot应用要直接与消息中间件进行信息交互的时候，由于各消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性。

- 通过定义绑定器作为中间层，完美地实现了**应用程序与消息中间件细节之间的隔离。**

- 通过向应用程序暴露统一的Channel通道， 使得应用程序不需要再考虑各种不同的消息中间件实现。

- **通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节之间的隔离。**

  

### Binder

在没有绑定器这个概念的情况下，我伽的SpringBoot应用要直接与消息中间件进行信息交互的吋候，由于各消息中向件构建的初衷不同,它们的实现细节上会有较大的差异性.**通过定义绑定器作为中间层，完美地实现了应用程序与消息中间件细节之间的隔离。**Stream对消息中间件的进一步封装，可以做到代码层面中间件的无感知，甚至于动态的切换中间件（rabbit切换为kafka），使得为服务开发的高度解耦，服务可以关注更多自己的业务流程。

INPUT对应于消费者，OUTPUT对应于生产者。

![image-20201127112058969](/assets/imgs/image-20201127112058969.png)

## Stream中的消息通信方式遵循了发布-订阅模式

Topic主题进行广播。

在kafka中就是Topic，在RabbitMQ就是Exchange。



## Spring Cloud Stream标准流程套路

- `Binder` 很方便的连接中间件，屏蔽差异
- `Channel` 通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置
- `Source和Sink` 简单的可理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接受消息就是输入

![image-20201127141030325](/assets/imgs/image-20201127141030325.png)

## 编码API和常用注解

![image-20201127141352636](/assets/imgs/image-20201127141352636.png)



# 案例说明

1.RabbitMQ环境已经OK

2.工程中新建三个子模块

- cloud-stream-rabbitmq-provider8801,作为生产者进行发消息模块
- cloud-stream-rabbitmq-consumer8802,作为消息接收模块
- cloud-stream-rabbitmq-consumer8803,作为消息接收模块



# 消息驱动之生产者

1.新建Module [cloud-stream-rabbitmq-provider8801](https://github.com/Silincee/springcloud2020/tree/main/cloud-stream-rabbitmq-provider8801)

2.[POM](https://github.com/Silincee/springcloud2020/blob/main/cloud-stream-rabbitmq-provider8801/pom.xml)

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

3.[YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-stream-rabbitmq-provider8801/src/main/resources/application.yml)

```yml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          output: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```

4.[主启动类StreamMQMain8801](https://github.com/Silincee/springcloud2020/blob/main/cloud-stream-rabbitmq-provider8801/src/main/java/cn/silince/springcloud/StreamMQMain8801.java)

5.业务类

- [发送消息接口](https://github.com/Silincee/springcloud2020/blob/main/cloud-stream-rabbitmq-provider8801/src/main/java/cn/silince/springcloud/service/IMessageProvider.java)
- [发送消息接口实现类](https://github.com/Silincee/springcloud2020/blob/main/cloud-stream-rabbitmq-provider8801/src/main/java/cn/silince/springcloud/service/impl/MessageProviderImpl.java)
- [Controller](https://github.com/Silincee/springcloud2020/blob/main/cloud-stream-rabbitmq-provider8801/src/main/java/cn/silince/springcloud/controller/SendMessageController.java)

6.测试

- 启动7001eureka
- 启动rabbitmq http://localhost:15672/
- 启动8801 http://localhost:8801/sendMessage

![image-20201127145017251](/assets/imgs/image-20201127145017251.png)



# 消息驱动之消费者

1.新建Module cloud-stream-rabbitmq-consumer8802

2.POM

3.YML

4.主启动类StreamMQMain8802

5.业务类

6.测试8801发送8802接收消息 http://localhost:8801/sendMessage

7.依照8802，clone出来一份运行cloud-stream-rabbitmq-consumer8803



# 分组消费与持久化

1.启动服务注册7001、消息生产8801、消息消费8802/8803

2.运行后两个问题

- **有重复消费问题**
- **消息持久化问题**

3.目前是8802/8803同时都收到了，存在重复消费问题



## 重复消费问题

目前是8802/8803同时都收到了，存在重复消费问题

***导致原因：***

- ***默认分组group是不同的，组流水号不一样，被认为不同组，可以消费。***

***如何解决？***

- 分组和持久化属性group

生产实际案例：

比如在如下场景中，订单系统我们做集群部署， 都会从RabbitMQ中获取订单信息，

**那如果一个订单同时被两个服务获取到，那么就会造成数据错误，**我们得避免这种情况。这时我们就可以使用**Stream中的消息分组来解决**

![image-20201127164828742](/assets/imgs/image-20201127164828742.png)

注意在Stream中处于同一个group中的多个消费者是竞争关系，就能够保证消息只会被其中一个应用消费一次。**不同组是可以全面消费的（重复消费），同一组内会发生竞争关系，只有其中一个可以消费。**



## 自定义分组

微服务应用放置于同一个group中，就能够保证消息只会被其中一个应用消费一次。**不同的组是可以消费的，同一个组内会发生竞争关系，只有其中一个可以消费。**



![image-20201127165446057](/assets/imgs/image-20201127165446057.png)

1.8802/8803都变成不同组，group两个不同

- 修改yml配置文件

8802:

```yml
        bindings: # 服务的整合处理
          input: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置
            group: silinceA
```

8803:

```yml
        bindings: # 服务的整合处理
          input: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置
            group: silinceB
```

![image-20201127180812537](/assets/imgs/image-20201127180812537.png)



2.8802/8803都变成相同组，group两个相同。都配置为`silinceA`

8802/8803实现了轮询分组，每次只有一个消费者 8801模块的发的消息只能被8802或8803其中一个接收到，这样避免了重复消费。

**同一个组的多个微服务实例，每次只会有一个拿到，默认轮询。**



## 持久化

1.停止8802/8803并去除掉8802的分组group:atguiguA（8803的分组group:atguiguA没有去掉）

2.8801先发送4条信息到rabbitmq

3.先启动8802，无分组属性配置，后台没有打出来消息

4.先启动8803，有分组属性配置，后台打出来了MQ上的消息