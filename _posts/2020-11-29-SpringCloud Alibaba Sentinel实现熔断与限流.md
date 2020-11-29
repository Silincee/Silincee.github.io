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

![image-20201129121458418](/assets/imgs/image-20201129121458418.png)

与Hystrix的区别：

![image-20201129120929766](/assets/imgs/image-20201129120929766.png)

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

![image-20201129124233657](/assets/imgs/image-20201129124233657.png)



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

![image-20201129124530532](/assets/imgs/image-20201129124530532.png)

## 流控模式

### 直接（默认）

直接 --->`QPS>1`快速失败配置：

![WX20201129-130423](/assets/imgs/WX20201129-130423.png)

表示1秒钟内查询1次就是0K，若超过次数1，就直接 ---> 快速失败，报默认错误。

- QPS指的是挡在外部不让进来
- 线程数指的是进来之后最多只能有 阈值n个线程执行，多的就快速失败

![image-20201129130524104](/assets/imgs/image-20201129130524104.png)

![image-20201129132557985](/assets/imgs/image-20201129132557985.png)



结果：

![image-20201129130620804](/assets/imgs/image-20201129130620804.png)

直接调用默认报错信息，技术方面OK but，是否应该有我们自己的后续处理？

- ***是否有类似于fallback的兜底方法？***



### 关联

- 当关联的资源达到阈值时，就限流自己
- 当与A关联的资源B达到阈值后，就限流自己
- 即B惹事，A挂了

1.配置/testA

配置效果：当关联资源/testB的qps阀值超过1时，就限流/testA的Rest访问地址，当关联资源到阈值后限制配置好的资源名

![image-20201129133142982](/assets/imgs/image-20201129133142982.png)

2.postman模拟并发密集访问testB

20个线程每隔间隔0.3s访问一次 /testB

![image-20201129135415786](/assets/imgs/image-20201129135415786.png)



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

![image-20201129144751921](/assets/imgs/image-20201129144751921.png)

源码：

![image-20201129154253425](/assets/imgs/image-20201129154253425.png)

Warmup配置案例：

阀值为10 + 预热时长设置5秒。

系统初始化的阀值为10 / 3约等于3，即阀值刚开始为3；然后过了5秒后阀值才慢慢升高恢复到10

![image-20201129155032695](/assets/imgs/image-20201129155032695.png)



### 排队等待

匀速排队，让请求以均匀的速度通过，阀值类型必须设成QPS，否则无效。**（只允许QPS）**

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景， 在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希盐系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

![image-20201129160516692](/assets/imgs/image-20201129160516692.png)

设置含义: /testA每秒1次请求，超过的话就排队等待，等待的超时时间为20000毫秒。

![image-20201129155440886](/assets/imgs/image-20201129155440886.png)

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

![image-20201129161329132](/assets/imgs/image-20201129161329132.png)



## 降级策略实战

### RT

慢调用比例 (`SLOW_REQUEST_RATIO`)：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。

![image-20201129162355620](/assets/imgs/image-20201129162355620.png)

配置：200ms内完成不了就触发熔断，经过熔断时长`1s`后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。

![image-20201129162616608](/assets/imgs/image-20201129162616608.png)

JMeter测试：1秒内打入10个线程

![image-20201129163844506](/assets/imgs/image-20201129163844506.png)

永远一秒钟打进来10个线程（大于5个了）调用testD，我们希望200毫秒处理完本次任务，

如果超过200毫秒还没处理完，在未来1秒钟的时间窗口内，断路器打开（保险丝跳闸）微服务不可用，保险丝跳闸断电了后续我停止jmeter，没有这么大的访问量了，断路器关闭（保险丝恢复），微服务恢复OK



### 异常比例

异常比例 (`ERROR_RATIO`)：当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。

![image-20201129164101131](/assets/imgs/image-20201129164101131.png)

配置：请求中的异常总数大于20%时候，触发熔断，时间窗口为3秒。

![image-20201129170158836](/assets/imgs/image-20201129170158836.png)

按照上述配置，单独访问一次，必然来一 次报错一次（int age = 10/0）， 调一次错一次；

开启jmeter后，直接高并发发送请求，多次调用达到我们的配置条件了。断路器开启（保险丝跳闸），微服务不可用了，不再报错error而是服务降级了。



### 异常数

异常数 (`ERROR_COUNT`)：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。

⚠️ ***时间窗口一定要大于等于60秒，异常是按分钟统计的。***

![image-20201129175239777](/assets/imgs/image-20201129175239777.png)

# [热点key限流](https://github.com/alibaba/Sentinel/wiki/热点参数限流)

## 基本介绍

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

![image-20201129184421585](/assets/imgs/image-20201129184421585.png)

Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。热点参数限流支持集群模式。



## 承上启下复习start

兜底方法

分为系统默认和客户自定义，两种：

之前的case，限流出问题后，都是用sentinel系统默认的提示: Blocked by Sentinel （flow limiting）

我们能不能自定？类似hystrix，某个方法出问题了，就找对应的兜底降级方法？

结论.

从HystrixCommand到@SentnelResource。



## 代码配置与测试

1.FlowLimitController中添加以下方法

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey") // value可为唯一的任意值
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2){
  return "-------testHotKey";
}

public String deal_testHotKey(String p1, String p2, BlockException exception){
  return "----deal_testHotKey,😈"; //sentinel系统默认的提示: Blocked by Sentinel （flow limiting）
}
```

2.新增热点规则

方法testHostKey里面第0个参数`p1`只要QPS超过每秒1次，马上降级处理`deal_testHotKey()`。

![image-20201129191336955](/assets/imgs/image-20201129191336955.png)

3.测试结果

![image-20201129191716190](/assets/imgs/image-20201129191716190.png)

## 参数例外项

上述案例演示了第一个参数p1,当QPS超过1秒1次点击后马上被限流

1.特殊情况

- 普通 超过1秒钟一个后，达到阈值1后马上被限流
- 我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样
- 特例：假如当p1的值等于5时，它的阈值可以达到200

2.配置，添加按钮不能忘

前提条件：热点参数的注意点，参数必须是基本类型或者String

⚠️ ***运行时的异常不会进行兜底，仍然会报错。***

![image-20201129193232237](/assets/imgs/image-20201129193232237.png)

## 注意事项

`@SentinelResource`处理的是Sent inel控制台配置的违规情况，有blockHandler方法配 置的兜底处理；

**RuntimeException(如int age = 10/0）**，这个是java运行时报出的运行时异常RunTimeException， @SentinelResource不管

***总结: `@SentinelResource`主管配置出错，运行出错该走异常走异常***



# [系统规则](https://github.com/alibaba/Sentinel/wiki/%E7%B3%BB%E7%BB%9F%E8%87%AA%E9%80%82%E5%BA%94%E9%99%90%E6%B5%81)

Sentinel 系统自适应限流从整体维度对**应用入口流量进行控制**，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

**系统保护规则**是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是应用整体维度的，而不是资源维度的，并且**仅对入口流量生效**。入口流量指的是进入应用的流量（`EntryType.IN`），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

系统规则支持以下的模式：

- **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
- **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
- **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
- **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
- **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。



# @SentinelResource

## 按资源名称限流 + 后续处理

1.启动Nacos和Sentinel成功

2.打开Module [cloudalibaba-sentinel-service8401](https://github.com/Silincee/springcloud2020/tree/main/cloudalibaba-sentinel-service8401)

- YML

```yml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        port: 8719 # 默认8719端口，加入被占用会从8719开始依次+1扫描，直至找到为被占用饿的端口

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

- 业务类RateLimitController

```java
public class RateLimitController {

    @GetMapping("/byResource")
    @SentinelResource(value = "byResource",blockHandler = "handleException")
    public CommonResult byResource(){
        return new CommonResult(200,"按资源名称限流测试ok",new Payment(2020L,"serial001"));
    }

    public CommonResult handleException(BlockException exception){
        return new CommonResult(444,exception.getClass().getCanonicalName()+"\t 服务不可用");
    }
}
```

3.配置流控规则

表示1秒钟内查询次数大于1，就跑到我们自定义的处流，限流

![image-20201129201329196](/assets/imgs/image-20201129201329196.png)

4.测试 http://localhost:8401/byResource

- 1秒钟点击1下，OK
- 超过上述问题，疯狂点击，返回了自己定义的限流处理信息，限流发送

## 额外问题

此时关闭微服务8401看看,Sentinel控制台，流控规则消失了？？？？？

临时/持久？



## 按照Url地址限流 + 后续处理

通过访问的URL来限流，会返回Sentinel自带默认的限流处理信息

- 业务类RateLimitController, `/byUrl`无兜底方法，将会采用系统默认的。

```java
@RestController
public class RateLimitController {

    @GetMapping("/byResource")
    @SentinelResource(value = "byResource",blockHandler = "handleException")
    public CommonResult byResource(){
        return new CommonResult(200,"按资源名称限流测试ok",new Payment(2020L,"serial001"));
    }

    public CommonResult handleException(BlockException exception){
        return new CommonResult(444,exception.getClass().getCanonicalName()+"\t 服务不可用");
    }

    @GetMapping("/rateLimit/byUrl")
    @SentinelResource(value = "byUrl")
    public CommonResult byUrl(){
        return new CommonResult(200,"按url限流测试ok",new Payment(2020L,"serial002"));
    }
}
```

- Sentinel控制台配置

![image-20201129202324589](/assets/imgs/image-20201129202324589.png)

- 测试结果，会返回sentiel自带的服务限流结果

![image-20201129202437317](/assets/imgs/image-20201129202437317.png)

## 上面兜底方法面临的问题

- 系统默认的，没有体现我们自己的业务要求。
- 依照现有条件，我们自定义的处理方法又和业务代码耦合在一块，不直观。
- 每个业务方法都添加一个兜底的，那代码膨胀加剧。
- 全局统一 的处理方法没有体现。



## 客户自定义限流处理逻辑

1.创建customerBlockHandler类用于自定义限流处理逻辑

2.自定义限流处理类 CustomerBlockHandler

```java
public class CustomerBlockHandler {

    public static CommonResult handlerException(BlockException exception){
        return new CommonResult(4444,"按客户自定义，global handlerException---1");
    }

    public static CommonResult handlerException2(BlockException exception){
        return new CommonResult(4444,"按客户自定义，global handlerException---2");
    }
}
```

3.RateLimitController

```java
@GetMapping("/rateLimit/customerBlockHandler")
@SentinelResource(value = "customerBlockHandler",
                  blockHandlerClass = CustomerBlockHandler.class,
                  blockHandler = "handlerException2")
public CommonResult customerBlockHandler(){
  return new CommonResult(200,"按客户自定义",new Payment(2020L,"serial003"));
}
```

4.启动微服务后先调用一次 http://localhost:8401/rateLimit/customerBlockHandler

5.Sentinel控制台配置

![image-20201129203833705](/assets/imgs/image-20201129203833705.png)

6.测试结果

![image-20201129203818937](/assets/imgs/image-20201129203818937.png)

7.解释说明

![image-20201129203931854](/assets/imgs/image-20201129203931854.png)



## [更多注解属性说明](https://github.com/alibaba/Sentinel/wiki/注解支持)

Sentinel主要有三个核心API(了解，一般不会用代码形式配置):

- SphU定义资源
- Tracer定义统计
- ContextUtil定义了上下文



# 服务熔断功能

1.sentinel整合ribbon+openFeign+fallback

2.启动nacos和sentinel

## 3.Ribbon系列

- 提供者9003/9004
  - 新建[cloudalibaba-provider-payment9003](https://github.com/Silincee/springcloud2020/tree/main/cloudalibaba-provider-payment9003)/[9004](https://github.com/Silincee/springcloud2020/tree/main/cloudalibaba-provider-payment9004)
  - 测试地址 http://localhost:9003/paymentSQL/1

- 消费者84

  - 新建[cloudalibaba-consumer-nacos-order84](https://github.com/Silincee/springcloud2020/tree/main/cloudalibaba-consumer-nacos-order84/src/main/java/cn/silince/springcloud/alibaba)

  - 业务类ApplicationContextConfig

  - 业务类CircleBreakerController的全部源码

    1）修改后请重启微服务。热部署对java代码级生效及时，对@SentinelResource注解内属性，有时效果不好

    2）目的：fallback管运行异常 / blockHandler管配置违规

    3）测试地址 http://localhost:84/consumer/fallback/1 轮训访问ok

    4）没有任何配置 给客户error页面，不友好  http://localhost:84/consumer/fallback/4

    5）只配置fallback。只负责业务异常

    6）只配置blockHandler。只负责sentinel控制台配置违规

    7）fallback和blockHandler都配置。***若blockHandler和fallback都进行了配置，则被限流降级且抛出BlockException时只会进入blockHandler处理逻辑。***

    8）异常忽略属性`exceptionsToIgnore`  

    ![image-20201129214848313](/assets/imgs/image-20201129214848313.png) 

```java
@RequestMapping("/consumer/fallback/{id}")
//@SentinelResource(value = "fallback") //没有配置
//@SentinelResource(value = "fallback",fallback = "handlerFallback") //fallback只负责业务异常
//@SentinelResource(value = "fallback",blockHandler = "blockHandler") //blockHandler只负责sentinel控制台配置违规
@SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler",exceptionsToIgnore = {IllegalArgumentException.class})
public CommonResult<Payment> fallback(@PathVariable Long id)
{
  CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/"+id, CommonResult.class,id);

  if (id == 4) {
    throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
  }else if (result.getData() == null) {
    throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
  }

  return result;
}
//本例是fallback
public CommonResult handlerFallback(@PathVariable  Long id,Throwable e) {
  Payment payment = new Payment(id,"null");
  return new CommonResult<>(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
}
//本例是blockHandler
public CommonResult blockHandler(@PathVariable  Long id,BlockException blockException) {
  Payment payment = new Payment(id,"null");
  return new CommonResult<>(445,"blockHandler-sentinel限流,无此流水: blockException  "+blockException.getMessage(),payment);
}
```



![image-20201129210045034](/assets/imgs/image-20201129210045034.png)

## 4.Feign系列

1）修改84模块。84消费者调用提供者9003，Feign逐渐一般是消费侧

- POM 引入

```xml
<!--SpringCloud openfeign -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

- YML

```yml
# 激活Sentinel对Feign的支持
feign:
  sentinel:
    enabled: true
```

- 主启动 添加@EnableFeignClients启动Feign的功能
- 业务类
  - [带@FeignClient注解的业务接口](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order84/src/main/java/cn/silince/springcloud/alibaba/service/PaymentService.java) fallback = PaymentFallbackService.class
  - [PaymentFallbackService实现类](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order84/src/main/java/cn/silince/springcloud/alibaba/service/PaymentFallbackService.java)
  - [Controller](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order84/src/main/java/cn/silince/springcloud/alibaba/controller/CircleBreakerController.java)

2）测试

- http://localhost:84/consumer/paymentSQL/1
- 测试84调用9003，**此时故意关闭9003微服务提供者**，看84消费侧自动降级，不会被耗死



## 熔断框架比较

|                | Sentinel                                                   | Hystrix               | resilience4j                     |
| -------------- | ---------------------------------------------------------- | --------------------- | -------------------------------- |
| 隔离策略       | 信号量隔离(并发现线程数限流)                               | 线程池隔离/信号量隔离 | 信号量隔离                       |
| 熔断降级策略   | 基于响应时间、异常比率、异常数                             | 基于异常比率          | 基于异常比率、响应时间           |
| 实时统计实现   | 滑动窗口(LeapArray)                                        | 滑动窗口(RxJava)      | Ring Bit Buffer                  |
| 动态规则配置   | 支持多种数据源                                             | 支持多种数据源        | 有限支持                         |
| 扩展性         | 多个扩展点                                                 | 插件的形式            | 接口的形式                       |
| 基于注解的支持 | 支持                                                       | 支持                  | 支持                             |
| 限流           | 基于QPS、支持基于调用关系的限流                            | 有限的支持            | Rate Limiter                     |
| 流量整形       | 支持预热模式、匀速器模式、预热排队模式                     | 不支持                | 简单的Rate Limiter模式           |
| 系统自适应保护 | 支持                                                       | 不支持                | 不支持                           |
| 控制台         | 提供开箱即用的控制台，可配置规则、查看秒级监控、机器发现等 | 简单的监控查看        | 不提供控制台，可对接其他监控系统 |





# 规则持久化

一旦我们重启应用，sentinel规则将消失，生产环境需要将配置规则进行持久化。

将限流配置规则持久化进Nacos保存，只要刷新8401某个rest地址， sentinel控制台的流控规则就能看到，只要Nacos里面的配置不删除，针对8401上的sentinel流控规则持续有效。

1.修改cloudalibaba-sentinel-service8401

2.POM

```xml
<!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
<dependency>
  <groupId>com.alibaba.csp</groupId>
  <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

3.YML

```yml
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        port: 8719 # 默认8719端口，加入被占用会从8719开始依次+1扫描，直至找到为被占用饿的端口
      datasource: # 添加Nacos数据源配置
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
```

4.添加Nacos业务规则配置

- resource : 资源名称； 
- limitApp:来源应用；
- grade:阈值类型，0表示线程数，1表示QPS；
- count:单机阈值；
- strategy:流控模式，0表示直接，1表示关联，2表示链路；
- controlBehavior:流控效果，0表示快速失败，1表示Warm Up， 2表示排队等待；
- clusterMode:是否集群。

![image-20201129223334063](/assets/imgs/image-20201129223334063.png)

5.启动8401后刷新sentinel发现业务规则有了

![image-20201129223506981](/assets/imgs/image-20201129223506981.png)

6.快速访问测试接口

7.停止8401再看sentinel

8.重新启动8401再看sentinel

- 乍一看还是没有，稍等一会儿
- 多次调用 http://localhost:8401/rateLimit/byUrl
- 重新配置出现了，持久化验证通过