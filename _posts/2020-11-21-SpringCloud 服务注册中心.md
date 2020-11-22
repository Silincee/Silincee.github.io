---
layout: post
title: "SpringCloud 服务注册中心"
date: 2020-11-21 19:21:16 +0800--
categories: [Java,]
tags: [Java, 分布式, SpringCloud]  

---

# Eureka服务注册与发现

## 基础知识

1.什么是服务治理

Spring Cloud封装了Netflix 公司开发的Eureka模块来实现服务治理

在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。



2.什么是服务注册

Eureka采用了CS的设计架构， Eureka Server作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用Eureka的客户端连接到Eureka Server并维持心跳连接。这样系统的维护人员就可以通过Eureka Server 来监控系统中各个微服务是否正常运行。

在服务注册与发现中，有一个注册中心。 当服务器启动的时候，会把当前自己服务器的信息比如服务地址通讯地址等以别名方式注册到注册中心上。另一方（消费者|服务提供者） ，以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想:在于注册中心，因为使用注册中心管理每个服务与服务之间的一一个依赖关系（服务治理概念。在任何rpc远程框架中，都会有一个注册中心（存放服务地址相关信息---接口地址）

![image-20201121213909447](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201121213909447.png)

***Eureka包含两个组件: Eureka Server和Eureka Client：***

- ***Eureka Server提供服务注册服务***：各个微服务节点通过配置启动后，会在EurekaServer中进行注册， 这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。
- ***EurekaClient通过注册中心进行访问***：是一个Java客户端，用于简化Eureka Server的交互，户端同时也具备一个内置的、 使用轮询（round一robin）负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳（默认周期为30秒）。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）



## 单机构建步骤

1）[IDEA生成eurekaServer端服务注册中心cloud-eureka-server7001](https://github.com/Silincee/springcloud2020/tree/main/cloud-eureka-server7001)  @EnableEurekaServer

2）[EurekaClient端cloud-provider-payment8001将注册进EurekaServer成为服务提供者provider](https://github.com/Silincee/springcloud2020/tree/main/cloud-provider-payment8001) @EnableEurekaClient

3）[EurekaClient端cloud-consumer-order80将注册进EurekaServer成为服务消费者consumer]()





## 集群构建步骤

1.集群原理说明

问题：微服务RPC远程服务调用最核心的是什么？

高可用，试想你的注册中心只有一个only one，它出故障了那就呵呵了， 会导致整个微服务环境不可用。

解决方案：搭建Eureka注册中心集群，实现负载均衡+故障容错

![image-20201122132749158](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201122132749158.png)

2.集群环境构建步骤

- 参考cloud-eureka-server7001,新建[cloud-eureka-server7002](https://github.com/Silincee/springcloud2020/tree/main/cloud-eureka-server7002)

- 改POM

- 修改映射配置，host文件

  ```shell
  # springcloud2020
  127.0.0.1	eureka7001.com
  127.0.0.1	eureka7002.com
  ```

- [写YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-eureka-server7002/src/main/resources/application.yml)

- 主启动(复制cloud-eureka-server7001的主启动类到7002即可)

3.将支付服务8001微服务发布到上面2台Eureka集群配置中 [YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8001/src/main/resources/application.yml)

4.将订单服务80微服务发布到上面2台Eureka集群配置中 [YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-order80/src/main/resources/application.yml)

5.支付服务提供者8001集群环境构建

- 参考cloud-provider-payment8001,新建[cloud-provider-payment8002](https://github.com/Silincee/springcloud2020/tree/main/cloud-provider-payment8002)
- [POM](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8002/pom.xml)
- [YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8002/src/main/resources/application.yml)
- [主启动类](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8002/src/main/java/cn/silince/springcloud/PaymentMain8002.java)
- 修改[8001](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8001/src/main/java/cn/silince/springcloud/controller/PaymentController.java)/[8002](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8002/src/main/java/cn/silince/springcloud/controller/PaymentController.java)的Controller，默认的方式为轮询

6.负载均衡

- ⚠️ [订单服务访问地址不能写死！](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-order80/src/main/java/cn/silince/springcloud/controller/OrderController.java)

  ```java
  public static final String PAYMENT_URL="http://CLOUD-PAYMENT-SERVICE";
  ```

  



## actuator微服务信息完善

## 服务发现Discovery

## eureka自我保护





# Zookeeper服务注册与发现





# Consul服务注册与发现