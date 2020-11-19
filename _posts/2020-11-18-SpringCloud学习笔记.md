---
layout: post
title:  "SpringCloud学习笔记"
date:   2020-11-18 19:21:16 +0800--
categories: [Java,]
tags: [Java, 分布式, ]  

---

# 微服务架构概述

服务架构是一种架构模式,它提倡将单一应用程序划分成一组小的服务,服务之间互相协调、互相配合,为用户提供最终价值。每个服务运行在其独立的进程中,服务与服务间采用轻量级的通信机制互相协作(通常是基于HTTP协议的 (RESTful api) 。每个服务都围绕着具本业务进行构建,并且能够被独立的部署到生产环境、类生产环境等。另外,应当尽量避免统一的、集中式的服务管理机制,对具体的一个服务而应根据业务上下文,选择合适的语言、工具对其进行构建。

![image-20201118220217597](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201118220217597.png)

涉及到的技术：

- 服务的注册与发现	
- 服务调用
- 服务熔断
- 负载均衡
- 服务降级
- 服务消息队列
- 配置中心管理
- 服务网关
- 服务监控
- 全链路追踪
- 自动化构建部署
- 服务定时任务调度操作

![image-20201118220108156](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201118220108156.png)



# 架构选型

同时用boot和cloud，需要照顾cloud，由cloud决定boot版本

SpringCloud和Springboot之间的依赖关系:https://spring.io/projects/spring-cloud#overview

- cloud	`Hoxton.SR1`
- boot	`2.2.2.RELEASE`
- cloud alibaba	`2.1.0.RELEASE`
- Java	`Java8`
- Maven	`>=3.5`
- Mysql	`>=5.7`



# 关于Cloud各种组件的停更/升级/替换

