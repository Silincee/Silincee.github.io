---
layout: post
title: "SpringCloud 服务注册中心"
date: 2020-11-21 19:21:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# Eureka服务注册与发现

> Eureka已停止更新：https://github.com/Netflix/eureka/wiki

## 基础知识

1.什么是服务治理

Spring Cloud封装了Netflix 公司开发的Eureka模块来实现服务治理

在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。



2.什么是服务注册

Eureka采用了CS的设计架构， Eureka Server作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用Eureka的客户端连接到Eureka Server并维持心跳连接。这样系统的维护人员就可以通过Eureka Server 来监控系统中各个微服务是否正常运行。

在服务注册与发现中，有一个注册中心。 当服务器启动的时候，会把当前自己服务器的信息比如服务地址通讯地址等以别名方式注册到注册中心上。另一方（消费者/服务提供者） ，以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想:在于注册中心，因为使用注册中心管理每个服务与服务之间的一一个依赖关系（服务治理概念。在任何rpc远程框架中，都会有一个注册中心（存放服务地址相关信息---接口地址）

![image-20201121213909447](/assets/imgs/image-20201121213909447.png)

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

![image-20201122132749158](/assets/imgs/image-20201122132749158.png)

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
- 修改[8001](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8001/src/main/java/cn/silince/springcloud/controller/PaymentController.java)/[8002](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8002/src/main/java/cn/silince/springcloud/controller/PaymentController.java)的Controller

6.负载均衡

- ⚠️ [订单服务访问地址不能写死！](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-order80/src/main/java/cn/silince/springcloud/controller/OrderController.java)

  ```java
  public static final String PAYMENT_URL="http://CLOUD-PAYMENT-SERVICE";
  ```

- [使用@LoadBalanced注解赋予RestTemplate负载均衡的能力](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-order80/src/main/java/cn/silince/springcloud/config/ApplicationContextConfig.java) ***Ribbon的负载均衡功能，默认的方式为轮询***

- Ribbon和Eureka整合后Consumer可以直接调用服务而不用再关心地址和端口号，且该服务还有负载功能了



## actuator微服务信息完善

1.服务显示格式`主机名称：服务名称修改`，需要去掉主机名称

2.使得访问信息有ip信息提示，方便排查

在`cloud-provider-payment8001`的`application.yml`添加以下配置：

```yml
eureka:
    instance-id: payment8001
    #访问路径可以显示IP地址
    prefer-ip-address: true
```

效果：

![WX20201122-155031](/assets/imgs/WX20201122-155031.png)



## 服务发现Discovery

***对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息***

1.修改`cloud-provider-payment8001`的Controller，添加以下内容

```java
@Autowired
private DiscoveryClient discoveryClient;

/**
 * @description: 对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息
 */
@GetMapping(value = "/payment/discovery")
public Object discovery(){
  // 方式一：获得eureka下所有的微服务名称列表
  List<String> services = discoveryClient.getServices();
  for (String service : services) {
    log.info("*****element: "+service);
  }

  // 方式二：根据微服务名称获得服务包含的实例列表
  List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
  for (ServiceInstance instance : instances) {
    log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
  }
  return this.discoveryClient;
}
```

2.8001主启动类添加`@EnableDiscoveryClient`注解

测试结果：

![image-20201122161621701](/assets/imgs/image-20201122161621701.png)



## eureka自我保护

1.故障现象

保护模式主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护。一旦进入保护模式，
***Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。***

如果在Eureka Server的首页看到以下这段提示，则说明Eureka进入了保护模式
![image-20201122161801553](/assets/imgs/image-20201122161801553.png)



2.导致原因

一句话：***某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存***，属于CAP里面的AP分支。

- 为什么会产生Eureka自我保护机制？

  为了防止EurekaClient可以正常运行，但是与EurekaServer网络不通情况下，EurekaServer**不会立刻将EurekaClient服务剔除**

- 什么是自我保护模式？

  默认情况下，如果EurekaServer在一定时间内没有接收到某 个微服务实例的心跳， EurekaServer将会注销该实例 （默认90秒）。但是当网络分区故障发生（延时、卡顿、拥挤）时， 微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了一因为微服务本身其实是健康的，**此时本不应该注销这个微服务**。Eureka通过 ”自我保护模式”来解决这个问题---当EurekaServer节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。

![image-20201122163733225](/assets/imgs/image-20201122163733225.png)

**在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。**

它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。**一句话讲解:好死不如赖活着。**

综上，自我保护模式是一种应对网络异常的安全保护措施。 它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留）也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。



3.怎么禁止自我保护（一般生产环境中不会禁止自我保护）

1）注册中心eureakeServer端7001

- 出厂默认，自我保护机制是开启的

- 使用`eureka.server.enable-self-preservation = false`可以禁用自我保护模式

  ![image-20201122170100824](/assets/imgs/image-20201122170100824.png)

2）生产者客户端eureakeClient端8001

- Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
  - `eureka.instance.lease-renewal-interval-in-seconds=30`  
- Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
  - `eureka.instance.lease-expiration-duration-in-seconds=90`  



# Zookeeper服务注册与发现

> Eureka停止更新了怎么办，使用SpringCloud整合Zookeeper代替Eureka
>
> [Zookerper学习笔记](http://www.silince.cn/2020/08/28/Zookeeper学习笔记/)

## 注册中心Zookeeper

- zookeeper是一个分布式协调工具，可以实现注册中心功能
- 关闭Linux服务器防火墙后启动zookeeper服务器
- zookeeper服务器取代Eureka服务器，zk作为服务注册中心

![image-20201122201938039](/assets/imgs/image-20201122201938039.png)

## 服务提供者

- 新建 [cloud-provider-payment8004](https://github.com/Silincee/springcloud2020/tree/main/cloud-provider-payment8004)

- [POM](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8004/pom.xml) 注意`spring-cloud-starter-zookeeper-discovery`中的jar包冲突

  ![image-20201122195923689](/assets/imgs/image-20201122195923689.png)

- [YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8004/src/main/resources/application.yml)

- [主启动类](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8004/src/main/java/cn/silince/springcloud/PaymentMain8004.java)

- [Controller](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8004/src/main/java/cn/silince/springcloud/controller/PaymentController.java)

测试结果：

![WX20201122-200615](/assets/imgs/WX20201122-200615.png)

🤔 服务节点是临时节点还是持久节点 ？

- ***是临时节点，服务未发送心跳直接去除***



## 服务消费者

> [Zookeeper集群配置](http://www.silince.cn/2020/08/28/Zookeeper学习笔记/#分布式安装)

- [新建cloud-consumerzk-order80](https://github.com/Silincee/springcloud2020/tree/main/cloud-consumerzk-order80)
- [POM](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumerzk-order80/pom.xml)
- [YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumerzk-order80/src/main/resources/application.yml)
- [主启动](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumerzk-order80/src/main/java/cn/silince/springcloud/OrderMainZK80.java)
- [业务类](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumerzk-order80/src/main/java/cn/silince/springcloud/controller/OrderZKController.java)
- 启动8004注册进zookeeper
- 验证测试，访问测试地址：http://localhost/consumer/payment/zk



# Consul服务注册与发现

## Consul简介

> 官网地址：https://www.consul.io/intro/index.html
>
> 文档：https://www.springcloud.cc/spring-cloud-consul.html

Consul是一开源的分布式服务发现和配置管理系统，由HashiCorp公司用Go语言开发。

提供了微服务系统中的服务治理、配置中心、控制总线等功能。这些功能中的每一个都可以根据需要 单独使用，也可以一起使用以构建全方位的服务网格，总之Consul提供了一种完整的服务网格解决方案。

它具有很多优点。包括:基于raft协议，比较简洁；支持健康检查， 同时支持HTTP和DNS协议支持跨数据中心的WAN集群提供图形界面跨平台，支持Linux、Mac、 Windows

能干嘛：

- 服务发现	
  - 提供HTTP和DNS两种发现方式
- 健康监测
  - 支持多种协议，HTTP、TCP、Docker、Shell脚本定制化
- KV存储
  - key , Value的存储方式
- 多数据中心
  - Consul支持多数据中心
- 可视化Web界面

## 安装并运行Consul

```shell
unzip consul_1.8.6_darwin_amd64.zip
mv consul ~/Applications/consul/bin
# 查看版本
consul --version
# 使用开发模式启动 
consul agent -dev
```

通过以下地址可以访问Consul的首页：http://localhost:8500，结果页面：![image-20201122211959858](/assets/imgs/image-20201122211959858.png)



## 服务提供者

- 新建Module支付服务[cloud-providerconsul-payment8006](https://github.com/Silincee/springcloud2020/tree/main/cloud-providerconsul-payment8006)
- POM
- YML
- 主启动类
- 业务类Controller
- 验证测试：http://localhost:8006/payment/consul



## 服务消费者

- 新建Module消费服务 [cloud-consumerconsul-order80](https://github.com/Silincee/springcloud2020/tree/main/cloud-consumerconsul-order80)
- POM
- YML
- 主启动类
- 配置Bean
- 业务类Controller
- 访问测试地址：http://localhost/consumer/payment/consul



# 小结

## 三个注册中心异同点

***CAP理论关注粒度是数据，而不是整体系统设计的策略***

- C:Consistency(强一致性)
- A:Availability(可用性)
- P:Partition tolerance(分区容错)

| 组件名    | 语言 | CAP  | 服务健康检查 | 对外暴露接口 | Spring Cloud集成 |
| --------- | ---- | ---- | ------------ | ------------ | ---------------- |
| Eureka    | Java | AP   | 可配支持     | HTTP         | 已集成           |
| Consul    | Go   | CP   | 支持         | HTTP/DNS     | 已集成           |
| Zookeeper | Java | CP   | 支持         | 客户端       | 已集成           |

**最多只能同时较好的满足两个。**

CAP理论的核心是: **一个分布式系统不可能同时很好的满足致性， 可用性和分区容错性这三个需求，**因此，根据CAP原理将NoSQL数据库分成了满足CA原则、满足CP原则和满足AP原则三大类:

- CA --- 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大
- CP --- 满足一致性， 分区容忍必的系统，通常性能不是特别高
- AP --- 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些

![image-20201122215359075](/assets/imgs/image-20201122215359075.png)

## AP架构

当网络分区出现后，为了保证可用性，**系统B可以返回旧值**，保证系统的可用性。

**结论:违背了一致性C的要求，只满足可用性和分区容错，即AP**

![image-20201122215918965](/assets/imgs/image-20201122215918965.png)



## CP架构

当网络分区出现后，为了保证一致性，就必须拒接请求，否则无法保证一致性

**结论:违背了可用性A的要求，只满足一致性和分区容错，即CP**

![image-20201122220112005](/assets/imgs/image-20201122220112005.png)

