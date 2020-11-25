---
layout: post
title: "SpringCloud 服务配置"
date: 2020-11-25 14:11:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# SpringCloud config分布式配置中心

## 概述

***分布式系统面临的配置问题：***

微服务意味着要将单体应用中的业务拆分成一个个子服务 ，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。

SpringCloud提供了ConfigServer来解决这个问题，我们每一 个微服务自己带着一个application.yml，上百个配置文件的管理。

***SpringCloud config 是什么：***

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。





