---
layout: post
title: "SpringCloud Alibaba"
date: 2020-11-27 19:18:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# SpringCloud Alibaba入门简介

## 为什么会出现SpringCloud alibaba

[Spring Cloud Netflix项目进入维护模式](https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now)

***什么是维护模式:***

将模块置于维护模式，意味着Spring Cloud团队将不会再向模块添加新功能。我们将修复block级别的bug以及安全问题，我们也会考虑并审查社区的小型pull request.

***进入维护模式意味着什么呢？***

Spring Cloud Netlix将不再开发新的组件 

我们都知道Spring Cloud版本迭代算是比较快的，因而出现了很多重大ISSUE都还来不及Fix就又推另一个Release了.进入维护模式意思就是目前一直以后 一段时间Spring Cloud Netflix提供的服务和功能就这么多了，不在开发新的组件和功能了。以后将以维护和Merge分支Full Request为主。

**新组件功能将以其他替代平代替的方式实现**



## SpringCloud alibaba是什么

***诞生:***

2018.10.31， [Spring Cloud Alibaba](https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md)正式入驻了Spring Cloud官方孵器，并在Maven中央库发布了第一个版本。

Spring Cloud for Alibaba，它是由一些阿里巴巴的开源组件和云产品组成的。 这个项目的目的是为了让大家所熟知的Spring框架，其优秀的设计模式和抽象理念，以给使用阿里巴巴产品的Java开发者带来使用Spring Boot和Spring Cloud的更多便利。

***功能：***

- 服务限流降级:默认支持Servlet. Feign. RestTemplate、 Dubbo 和RocketMQ限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级Metrics监控。
- 服务注册与发现:适配Spring Cloud服务注册与发现标准，默认集成了Ribbon的支持。
- 分布式配置管理:支持分布式系统中的外部化配置，配置更改时自动刷新。
- 消息驱动能力:基于Spring Cloud Stream为微服务应用构建消息驱动能力。
- 阿里云对象存储:阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- 分布式任务调度:提供秒级、精准、可靠、高可用的定时（基于Cron表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子务均均分配到所有Worker （schedulerx-client）上执行。

***怎么玩：***

![image-20201127201408218](/assets/imgs/image-20201127201408218.png)

***学习资料：***

- 官网：https://spring.io/projects/spring-cloud-alibaba#overview
- 英文：https://github.com/alibaba/spring-cloud-alibaba / https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html
- 中文：https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md



# Nacos服务注册和配置中心

> [中文文档](https://nacos.io/zh-cn/index.html)			 [英文文档](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_discovery)			[下载地址](https://github.com/alibaba/nacos/releases/tag/1.1.4)

## Nacos简介

为什么叫Nacos：前四个字母分别为Naming和Configuration的前两个字母，最后的s为Service。

Nacos是一个更易于构建云原生应用的动态服务发现，配置管理和服务管理中心。Nacos就是注册中心+配置中心的组合，等价于 `Eureka+Config+Bus`。可以替代Eureka做服务注册中心，替代Config做服务配置中心。

各种注册中心比较:

![image-20201127205627362](/assets/imgs/image-20201127205627362.png)

据说Nacos在阿里巴巴内部有超过10万的实例运行，已经过了类似双十一等各种大型流量的考验。



## 安装并运行Nacos

- 解压安装包，直接运行bin目录下的`startup.sh`
- 命令运行成功后直接访问http://localhost:8848/nacos/#/login
- 默认账号密码都是nacos

```shell
cd /Users/silince/Applications/Nacos/nacos/bin
sh startup.sh -m standalone
```

![image-20201127211652219](/assets/imgs/image-20201127211652219.png)

## Nacos作为服务注册中心演示

### 基于Nacos的服务提供者

1.新建Module [cloudalibaba-provider-payment9001](https://github.com/Silincee/springcloud2020/tree/main/cloudalibaba-provider-payment9001)

2.POM

- 父POM

  ```xml
  <!--spring cloud alibaba 2.1.0.RELEASE-->
  <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.1.0.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
  </dependency>
  ```

- 本模块POM

  ```xml
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
  ```

3.[YML](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-provider-payment9001/src/main/resources/application.yml)

```yml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

4.[主启动](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-provider-payment9001/src/main/java/cn/silince/springcloud/PaymentMain9001.java)

5.[业务类](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-provider-payment9001/src/main/java/cn/silince/springcloud/controller/PaymentController.java)

6.测试

- http://localhost:9001/payment/nacos/1

- nacos控制台

- 为了下一章节演示nacos的负载均衡，参照9001新建`cloudalibaba-provider-payment9002`

  ![image-20201127214141788](/assets/imgs/image-20201127214141788.png)



### 基于Nacos的服务消费者

1.新建Module [cloudalibaba-consumer-nacos-order83](https://github.com/Silincee/springcloud2020/tree/main/cloudalibaba-consumer-nacos-order83)

2.[POM](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order83/pom.xml)

3.[YML](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order83/src/main/resources/application.yml)

4.[主启动](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order83/src/main/java/cn/silince/springcloud/alibaba/OrderNacosMain83.java)

5.业务类

- [ApplicationContextBean](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order83/src/main/java/cn/silince/springcloud/alibaba/config/ApplicationContextConfig.java)	`@LoadBalance`
- [OrderNacosController](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order83/src/main/java/cn/silince/springcloud/alibaba/controller/OrderNacosController.java)

6.测试

- nacos控制台
- http://localhost:83/consumer/payment/nacos/13  83访问9001/9002，轮询负载OK





### 服务注册中心对比

![image-20201127222043334](/assets/imgs/image-20201127222043334.png)

#### Nacos和CAP

![image-20201127221922015](/assets/imgs/image-20201127221922015.png)

#### AP和CP模式的切换

**C是所有节点在同一 时间看到的数据是一 致的；而A的定义是所有的请求都会收到响应。**

何时选择使用何种模式？

- **如果不需要存储服务级别的信息且服务实例是通过`nacos-client`注册， 并能够保持心跳上报，那么就可以选择AP模式。**当前主流的服务如Spring cloud和Dubbo服务，都适用于AP模式，AP模式为了服务的可用性而减弱了一致性， 因此AP模式下只支持注册临时实例。
- **如果需要在服务级别编辑或者存储配置信息，那么CP是必须**，K8S服务和DNS服务则适用于CP模式。
- **CP模式下则支持注册持久化实例**，此时则是以Raft协议为集群运行模式，**该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。**

`curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'`



## Nacos作为服务配置中心演示

### Nacos作为配置中心-基础配置





### Nacos作为配置中心-分类配置

## Nacos集群和持久化配置 🤔