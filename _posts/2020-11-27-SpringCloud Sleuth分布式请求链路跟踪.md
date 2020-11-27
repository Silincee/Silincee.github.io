---
layout: post
title: "SpringCloud Sleuth分布式请求链路跟踪"
date: 2020-11-27 18:18:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# 概述

> 官网：https://github.com/spring-cloud/spring-cloud-sleuth

Spring Cloud Sleuth提供了一套完整的服务跟踪的解决方案，在分布式系统中提供追踪解决方案并且兼容支持了zipkin。

![image-20201127184230096](/assets/imgs/image-20201127184230096.png)

在微服务框架中，一个由客户端发起的请求在后端系统中会经过多 个不同的的服务节点调用来协同产生最后的请求结果，每一个前段求都会形成一复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。

![image-20201127183819444](/assets/imgs/image-20201127183819444.png)

# 搭建链路监控步骤

## 1.zipkin

SpringCloud从F版起已不需要自己构建Zipkin server了，只需要调用jar包即可。[zipkin-server-2.12.9.exec.jar 下载地址](https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/)

- 运行jar: `java -jar zipkin-server-2.12.9-exec.jar`
- 运行控制台：http://localhost:9411/zipkin/

![image-20201127185557499](/assets/imgs/image-20201127185557499.png)



**完整的调用链路**：表示一请求链路， 一条链路通过Trace ld唯标识 ，Span标识发起的请求信息，各span通过parent id关联起来。

- Trace：类似于树结构的Span集合，表示一条调用链路，存在唯一标示
- span：表示调用链路来源，通俗的理解span就是一次请求信息

![image-20201127185842897](/assets/imgs/image-20201127185842897.png)





## 2.服务提供者 [cloud-provider-payment8001](https://github.com/Silincee/springcloud2020/tree/main/cloud-provider-payment8001)

- POM

  ```xml
  <!--包含了sleuth+zipkin-->
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
  </dependency>
  ```

- YML

  ```yml
  # 新增 sleuth 配置
  spring:
    zipkin:
      base-url: http://localhost:9411
    sleuth:
      sampler:
      probability: 1 #采样率值介于 0 到 1 之间，1 则表示全部采集
  ```

- 业务类PaymentController

  ```java
  /** 
   * @description: zipkin链路调用测试
   */
  @GetMapping("/payment/zipkin")
  public String paymentZipkin(){
    return "Here is payment zipkin server fall back😈";
  }
  ```



## 3.服务消费者（调用方）[cloud-consumer-order80](https://github.com/Silincee/springcloud2020/tree/main/cloud-consumer-order80)

- POM

  ```xml
  <!--包含了sleuth+zipkin-->
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
  </dependency>
  ```

- YML

  ```yml
  # 新增 sleuth 配置
  spring:
    application:
      name: cloud-order-service
    zipkin:
      base-url: http://localhost:9411
    sleuth:
      sampler:
        probability: 1
  ```

- 业务类OrderController

  ```java
  /**
   * @description: zipkin + sleuth
   */
  @GetMapping("/consumer/payment/zipkin")
  public String paymentZipkin(){
    String result = restTemplate.getForObject("http://localhost:8001" + "/payment/zipkin/", String.class);
    return result;
  }
  ```

## 4.依次启动eureka7001/8001/80

80调用8001几次测试下 http://localhost/consumer/payment/zipkin

## 5.打开浏览器访问

 http://localhost:9411

![image-20201127193201547](/assets/imgs/image-20201127193201547.png)

![image-20201127193212287](/assets/imgs/WX20201127-193307.png)

查看依赖关系：

![WX20201127-193307](/assets/imgs/WX20201127-193307-6476835.png)

