---
layout: post
title: "SpringCloud Alibaba Sentinel实现熔断与限流"
date: 2020-11-29 11:18:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# 概述

> [官网](https://github.com/alibaba/Sentinel)	[中文官网](https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D)	[文档](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_sentinel)	[下载地址](https://github.com/alibaba/Sentinel/releases)

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- **完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

![image-20201129121458418](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129121458418.png)

与Hystrix的区别：

![image-20201129120929766](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129120929766.png)

# 安装Sentinel控制台

sentinel氛围两个组成：

- 核心库（Java客户端）不依赖任何框架/库，能够运行于所有Java运行时环境，同时对Dubbo/Spring Cloud等框架也有较好的支持。
- 控制台（Dashboard） 基于Spring Boot开发，打包后可以直接运行，不需要额外的Tomcat等应用容器。

安装步骤：

- 执行命令 `java -jar sentinel-dashboard-1.7.0.jar `
- 访问sentinel管理界面 http://localhost:8080 ，登录账号密码均为sentinel



# 初始化演示工程

1.启动Nacos8848成功 http://localhost:8848/nacos/#/login

2.新建Module [cloudalibaba-sentinel-service8401](https://github.com/Silincee/springcloud2020/tree/main/cloudalibaba-sentinel-service8401)

3.启动Sentinel8080

4.启动8401微服务查看sentienl控制台

- Sentinel采用懒加载机制，需要先执行一次访问
- http://localhost:8401/testA  / http://localhost:8401/testB

![image-20201129124233657](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129124233657.png)



# 流控规则

## 基本介绍

- 资源名:唯一名称，默认请求路径
- 针对来源: Sentine可以针对调用者进行限流，填写微服务名，默认default （不区分来源）
- 阈值类型/单机阈值:
  - QPS （每秒钟的请求数量） :当调用该api的QPS达到阈值的时候，进行限流
  - 线程数:当调用该api的线程数达到阈值的时候，进行限流
- 是否集群:不需要集群
- 流控模式: 
  - 直接: api达到限流条件时，直接限流
  - 关联:当关联的资源达到阈值时，就限流自己
  - 链路:只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流） 【api级别的针对来源】
- 流控效果:
  - 快速失败:直接失败，抛异常
  - Warm Up:根据codeFactor （冷加载因子，默认3）的值，从阈值/codeFactor， 经过预热时长，达到设置的QPS阈值
  - 排队等待：均速派对，让请求以均速的速度通过，阈值类型必须设置为QPS，否则无效

![image-20201129124530532](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129124530532.png)

## 流控模式

### 直接（默认）

直接 --->`QPS>1`快速失败配置：

![WX20201129-130423](/Users/silince/Develop/博客/blog_to_git/assets/imgs/WX20201129-130423.png)

表示1秒钟内查询1次就是0K，若超过次数1，就直接 ---> 快速失败，报默认错误。

- QPS指的是挡在外部不让进来
- 线程数指的是进来之后最多只能有 阈值n个线程执行，多的就快速失败

![image-20201129130524104](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129130524104.png)

![image-20201129132557985](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129132557985.png)



结果：

![image-20201129130620804](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129130620804.png)

直接调用默认报错信息，技术方面OK but，是否应该有我们自己的后续处理？

- ***是否有类似于fallback的兜底方法？***



### 关联

- 当关联的资源达到阈值时，就限流自己
- 当与A关联的资源B达到阈值后，就限流自己
- 即B惹事，A挂了

1.配置/testA

配置效果：当关联资源/testB的qps阀值超过1时，就限流/testA的Rest访问地址，当关联资源到阈值后限制配置好的资源名

![image-20201129133142982](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129133142982.png)

2.postman模拟并发密集访问testB

20个线程每隔间隔0.3s访问一次 /testB

![image-20201129135415786](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129135415786.png)



### 链路

只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流） ***[api级别的针对来源]***



## 流控效果

### 快速失败（默认的流控处理）

直接失败，抛出异常 `Blocked by Sentinel (flow limiting)`

源码：com.alibaba.csp.sentinel.slots.block.flow.controller.DefaultController

### [预热](https://github.com/alibaba/Sentinel/wiki/限流---冷启动)

当流量突然增大的时候，我们常常会希望系统从空闲状态到繁忙状态的切换的时间长一些。即如果系统在此之前长期处于空闲的状态，我们希望处理请求的数量是缓步的增多，经过预期的时间以后，到达系统处理请求个数的最大值。Warm Up（冷启动，预热）模式就是为了实现这个目的的。

Warm Up （ `RuleConstant.CONTROL_BEHAVIOR_WARM_UP` ）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通5过“冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间， 避免冷系统被压垮。

***应用场景：秒杀系统在开启的瞬间，会有很多流量上来，很有可能把系统打死，预热方式就是把为了保护系统，可慢慢的把流量放进来，慢慢的把阀值增长到设置的阀值。***

**公式：阈值除以coldFactor（默认值为3），经过预热时长后才会达到阈值。**

**默认coldFactor为3，即请求QPS从`threshold/3`开始，经预热时长逐渐升至设定的QPS阈值。**

![image-20201129144751921](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129144751921.png)

源码：

![image-20201129154253425](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129154253425.png)

Warmup配置案例：

阀值为10 + 预热时长设置5秒。

系统初始化的阀值为10 / 3约等于3，即阀值刚开始为3；然后过了5秒后阀值才慢慢升高恢复到10

![image-20201129155032695](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129155032695.png)



### 排队等待

匀速排队，让请求以均匀的速度通过，阀值类型必须设成QPS，否则无效。**（只允许QPS）**

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景， 在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希盐系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

![image-20201129160516692](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129160516692.png)

设置含义: /testA每秒1次请求，超过的话就排队等待，等待的超时时间为20000毫秒。

![image-20201129155440886](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129155440886.png)

# 降级规则

官网：[https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7](https://github.com/alibaba/Sentinel/wiki/熔断降级)

## 概述

Sentinel熔断降级会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高）， 对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。

当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出DegradeException）。



### 熔断策略

***RT （平均响应时间，秒级）***

- 平均响应时间**超出阈值**且**在时间窗内通过的请求 >=5**，两个条件同时满足后触发降级
- 窗口期过后关闭断路器
- RT最大4900 （更大的需要通过 `-Dcsp.sentinel.statistic.max.rt= XXXX`才能生效）

***异常比例（秒级）***

- QPS >= 5且异常比例（秒级统计）超过阈值时，触发降级；时间窗结束后，关闭降级

***异常数（分钟级）***

- 异常数（分钟统计）超过阈值时，触发降级；时间窗口结束后，关闭降级

![image-20201129161329132](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129161329132.png)



## 降级策略实战

### RT

慢调用比例 (`SLOW_REQUEST_RATIO`)：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。

![image-20201129162355620](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129162355620.png)

配置：200ms内完成不了就触发熔断，经过熔断时长`1s`后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。

![image-20201129162616608](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201129162616608.png)



### 异常比例





### 异常树