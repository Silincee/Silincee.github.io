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

![image-20201127110714707](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201127110714707-6446772.png)

### 为什么用Cloud Stream

比方说我们用到了RabbitMQ和Kafka，由于这两个消息中间件的架构上的不同，像RabbitMQ有exchange， kafka有 Topic和Partitions分区。

这些中间件的差异性导致我们实际项目开发给我们造成了一定的困扰，我们如果用了两个消息队列的其中一种，后面的业务需求，我想往另外一种消息队列进行迁移，这时候无疑就是一个灾难性的， **一大堆东西都要重新推倒重新做**，因为它跟我们的系统耦合了，这时候springcloud Stream给我们提供了一种解耦合的方式。

![image-20201127110959509](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201127110959509-6446740.png)

### stream凭什么可以统一底层差异

在没有绑定器这个概念的情况下，我们的SpringBoot应用要直接与消息中间件进行信息交互的时候，由于各消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性。

- 通过定义绑定器作为中间层，完美地实现了**应用程序与消息中间件细节之间的隔离。**

- 通过向应用程序暴露统一的Channel通道， 使得应用程序不需要再考虑各种不同的消息中间件实现。

- **通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节之间的隔离。**

  

### Binder

在没有绑定器这个概念的情况下，我伽的SpringBoot应用要直接与消息中间件进行信息交互的吋候，由于各消息中向件构建的初衷不同,它们的实现细节上会有较大的差异性.**通过定义绑定器作为中间层，完美地实现了应用程序与消息中间件细节之间的隔离。**Stream对消息中间件的进一步封装，可以做到代码层面中间件的无感知，甚至于动态的切换中间件（rabbit切换为kafka），使得为服务开发的高度解耦，服务可以关注更多自己的业务流程。

![image-20201127112058969](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201127112058969.png)