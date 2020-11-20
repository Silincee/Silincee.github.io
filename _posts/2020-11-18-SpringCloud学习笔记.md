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

## 由停更引发的“升级惨案

停更不停用：

- 被动修复bugs
- 不再接受合并请求
- 不再发布新版本

以前：

![image-20201118220108156](/Users/silince/Pictures/Typora/image-20201118220108156-5785517.png)

now2020:

![image-20201119193752546](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201119193752546.png)





## 参考资料见官网

Spring Cloud：https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/

Spring Cloud中文文档：https://www.bookstack.cn/read/spring-cloud-docs/docs-index.md

Spring Boot：https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/htmlsingle/



# 微服务架构编码构建

## IDEA新建project工作空间

### 约定>配置>编码

### 父工程POM 

[pom.xml](https://github.com/Silincee/springcloud2020/blob/main/pom.xml)

### Maven中的dependencyManagement和dependencies

Maven使用dependencyManagement元素来提供了一种管理依赖版本号的方式。通常会在一个组织或者项目的最顶层的父POM中看到dependencyManagement元素。

⚠️注意：

- **dependencyManagement里只是声明依赖，并不实现，因此子项目需要显示声明需要用的依赖。**
- **如果不在子项目中声明赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom**
- **如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。**

使用`pom.xml`中的`dependencyManagement`元素能让所有在子项目中引用一个依赖而不用显式的列出版本号。Maven会沿着父子层次向上走，直到找到一个拥有`dependencyManagement`元素的项目，然后它就会使用这个`dependencyManagement`元素中指定的版本号。

例如在父项目里：

```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>mysql</groupId >
			<artifactId>mysql-connector-java</artifactId> 
			<version>5.1.2</version>
		</dependency>
		...
	<dependencies>
</dependencyManagement>
```

然后在子项目里就可以添加`mysql-connector`时可以不指定版本号，例如:

***这样做的好处就是:如果有多个子项目都引用同一样依赖，则可以避免在每个使用的子项目里都声明一个版本号，这样当想升级或切换到另一个版本时，只需要在顶层父容器里更新，而不需要一个个子项目的修改；另外如果某个子项目需要另外的一个版本，只需要声明version就可。***

```xml
<dependencies>
	<dependency>
		<groupId>mysql</groupId >
		<artifactId>mysql-connector-java</artifactId> 
	</dependency>
<dependencies>
```

## Rest微服务工程构建

### cloud-provider-payment8001微服务提供者支付Module模块

- [建module](https://github.com/Silincee/springcloud2020/tree/main/cloud-provider-payment8001)
- [改POM](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8001/pom.xml)
- [写YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8001/src/main/resources/application.yml)
- [主启动](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8001/src/main/java/cn/silince/springcloud/PaymentMain8001.java)
- 业务类



### 热部署Devtools

(⚠️只允许在开发阶段配置！)

（1）Adding devtools to your project

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <scope>runtime</scope>
  <optional>true</optional>
</dependency>
```

（2）Adding plugin to your pom.xml（父总工程）

```xml
<plugins>
  <plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
      <fork>true</fork>
      <addResources>true</addResources>
    </configuration>
  </plugin>
</plugins>
```

（3）Enabling automatic build 

![WX20201120-220714](/Users/silince/Develop/博客/blog_to_git/assets/imgs/WX20201120-220714.png)

（4）Update the value of

press `command+shift+Alt+/` and search for the registry.

![WX20201120-221226](/Users/silince/Develop/博客/blog_to_git/assets/imgs/WX20201120-221226-5881652.png)

（5）重启IDEA



### cloud-consumer-order80微服务消费者订单Module模块

#### RestTemplate

RestTemplate提供了多种便捷访问远程Http服务的方法，是一种简单便捷的访问restful服务模板类， 是Spring提供的用于访问Rest服务的***客户端模板工具集***。[官网地址](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html)


使用restTemplate访问restful接口非常的简单粗暴无脑。`url`， `requestMap`， `ResponseBean.class`这三个参数分别代表REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。

使用时候需要提供config配置类`ApplicationContextConfig.java`:

```java
/** 
* @description:  RestTemplate 配置类
*/ 
@Configuration
public class ApplicationContextConfig
{
    @Bean // 相当于在applicationContext.xml中配置了 <bean id="" class="">
    public RestTemplate getRestTemplate()
    {
        return new RestTemplate();
    }
}
```





### 工程重构