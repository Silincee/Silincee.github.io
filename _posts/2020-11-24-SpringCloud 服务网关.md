---
layout: post
title: "SpringCloud 服务网关"
date: 2020-11-24 12:11:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# Zuul

> 官网：https://github.com/Netflix/zuul/wiki

因为内部已出现了巨大分歧，已不推荐使用。 停止更新进入维护

Zuul2多次跳票。 研发中



# Gateway新一代网关

> 官网：https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/

## 概述简介

⭐️ ***一句话：Spring Cloud Gateway 使用的Webflux中的`reactor-netty`响应式编程组件，底层使用了Netty通讯框架。***

GateWay能干嘛：

- 反向代理
- 鉴权
- 流量控制
- 熔断
- 日志监控

Cloud全家桶中有个很重要的组件就是网关，在1.x版本中都是采用的Zuu网关；但在2.x版本中，zuu的升级直跳票， SpringCloud最后自己研发了一个网关替代Zuul，那就是SpringCloud Gateway一句话: **gateway是原zuul1 .x版的替代**

Gateway是在Spring生态系统之上构建的API网关服务，基基Spring 5， Spring Boot 2和Project Reactor等技术。

**Gateway旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能， 例如: 熔断、限流、重试等。**

SpringCloud Gateway是Spring Cloud的一个全新项目，基于Spring 5.0+ Spring Boot 2.0和Project Reactor等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的API路由管理方式。

SpringCloud Gateway作为Spring Cloud生态系统中的网关，目标是替代Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 1.x非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，***而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。***

Spring Cloud Gateway的目标提供统一的路由方式且基于Filter链的方式提供了网关基本的功能，例如:安全，监控/指标，和限流。



## 为什么选择GateWay

1.neflix不太靠谱，zuul2.0一直跳票,迟迟不发布

一方面因为Zuul1.0已经进入了维护阶段，且Gateway是SpringCloud团队研发的，亲儿子产品，值得信赖。且很多功能Zuul都没有用起来也非常的简单便捷。

**Gateway是基于异步非阻塞模型上进行开发的，性能方面不需要担心。**虽然Netflix早就发布 了最新的Zuul 2.x，但Spring Cloud貌似没有整合计划。而且Netflix相关组件都宣布进入维护期；不知前景如何？

多方面综合考虑Gateway是很理想的网关选择。



2.SpringCloud Gateway具有如下特性

- **基于Spring Framework 5， Project Reactor和Spring Boot 2.0进行构建；**
- 动态路由:能够匹配任何请求属性；
- 可以对路由指定Predicate （断言）和Filter （过滤器） ；
- 集成Hystrix的断路器功能； 
- 集成Spring Cloud服务发现功能；
- 易于编写的Predicate （断言）和Filter （过滤器） ；
- 请求限流功能；
- 支持路径重写。



3.SpringCloud Gateway与Zuul的区别

在 SpringCloud Finchley正式版之前, Spring Cloud推荐的网关是 Netflix提供的zuul:

- zuul1.x,是一个基于阻塞 l/O 的 API Gateway
- **Zuul1.x基于 Servlet 2.5使用阻塞架构**它不支持任何长连接(如WebSocket)Zuul设计模式和 Nginx较像,每次操作都是从工作线程中选择一个执行,请求线程被阻塞到工作线程完成,但是差别是 Nginx用C++实现,zuul用 Java实现,而 JVM 本身会有第一次加载较慢的情况，使得Zuul的性能相对较差。
- **Zuul2.x理念更先进,想基于 Netty非阻塞和支持长连接**,但 SpringCloud目前还没有整合zuul2.x的性能较Zuul 1.x有较大提升。在性能方面,根据官方提供的基准测试, Spring Cloud Gateway的RPS(每秒请求数)是Zuul的1.6倍。
- Spring Cloud Gateway建立在 Spring Framework5、 Project Reactor和 Spring Boot2之上,**使用非阻塞APl**
- Spring Cloud Gateway还支持 WebSocket,并且与 Spring紧密集成拥有更好的开发体验



4.Zuul1.x模型

Springcloud中所集成的Zuul版本，采用的是Tomcat容器，使用的是传统的Servlet IO处理模型。

**servlet由servlet container进行生命周期管理:**

- container启动时构造servlet对象并调用servlet init0进行初始化；
- container运行时接受请求，并为每个请求分配一个线程 （一般从线程池中获取空闲线程）然后调用`service()`。
- container关闭时调用servlet destory0销毁servlet；



5.GateWay模型







## 三大核心概念





## Gateway工作流程

## 入门配置

## 通过微服务名实现动态路由

## Predicate的使用

## Filter的使用