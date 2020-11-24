---
layout: post
title: "SpringCloud 服务降级Hystrix"
date: 2020-11-23 19:46:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# 分布式系统的服务雪崩

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。

![image-20201123195006080](/assets/imgs/image-20201123195006080.png)



多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的"**扇出**”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的”**雪崩效应**”.

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

所以，**通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。**



# Hystrix概述

> 官网资料：https://github.com/Netflix/Hystrix/wiki/How-To-Use
>
> 该项目目前已经停止更新，进入到了维护阶段

Hystrix是一个用于处理分布式系统的**延迟和容错的开源库**， 在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下， **不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。**

”断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝） ，**向调用方返回一个符合预期的、可处理的备选响应（FallBack） ， 而不是长时间的等待或者抛出调用方无法处理的异常**，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。



主要功能：

- 服务降级
- 服务熔断
- 接近实时的监控



# Hystrix重要概念

## 服务降级

服务器忙，请稍候再试，不让客户端等待并立刻返回一个友好提示，fallback

哪些情况会触发降级：

- 程序运行异常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量打满也会导致服务降级



## 服务熔断

类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电。然后调用服务降级的方法并返回友好提示

服务的降级->进而熔断->恢复调用链路



## 服务限流

秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行



# hystrix案例

## 环境构建

1.[新建cloud-provider-hystrix-payment8001](https://github.com/Silincee/springcloud2020/tree/main/cloud-provider-hystrix-payment8001)

2.[POM](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-hystrix-payment8001/pom.xml)

3.[YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-hystrix-payment8001/src/main/resources/application.yml)

4.[主启动](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-hystrix-payment8001/src/main/java/cn/silince/springcloud/PaymentHystrixMain8001.java)

5.业务类

- [service](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-hystrix-payment8001/src/main/java/cn/silince/springcloud/service/impl/PaymentServiceImpl.java)
- [controller](https://github.com/Silincee/springcloud2020/tree/main/cloud-provider-hystrix-payment8001/src/main/java/cn/silince/springcloud/controller)

6.正常测试

- 启动eureka7001
- 启动cloud-provider-hystrix-payment8001
- 访问
  - 成功的方法：http://localhost:8001/payment/hystrix/ok/31
  - 每次调用耗费5秒钟：http://localhost:8001/payment/hystrix/timeout/31
- 上述module均OK，以上述为根基平台，从正确->错误->降级熔断->恢复



## 高并发测试

上述在非高并发情形下，还能勉强满足  but.....

### Jmeter压测测试：

1.开启Jmeter，来20000个并发压死8001，20000个请求都去访问paymentInfo_TimeOut服务

```shell
cd /Users/silince/Applications/jmeter/apache-jmeter-5.3/bin
./jmeter.sh
```

![image-20201123204311947](/assets/imgs/image-20201123204311947.png)

![image-20201123204542479](/assets/imgs/image-20201123204542479.png)

![image-20201123204646299](/assets/imgs/image-20201123204646299.png)

![WX20201123-204845](/assets/imgs/WX20201123-204845.png)

2.再来一个访问

- http://localhost:8001/payment/hystrix/ok/31
- http://localhost:8001/payment/hystrix/timeout/31

3. 看演示结果

- 两个都在自己转圈圈，连坐效应
- 为什么会被卡死
  - tomcat的默认的工作线程数被打满了，没有多余的线程来分解压力和处理。

***Jmeter压测结论：***

**上面还是服务提供者8001自己测试，假如此时外部的消费者80也来访问，那消费者只能干等，最终导致消费端80不满意，服务端8001直接被拖死**



### 看热闹不嫌弃事大，80新建加入

1.新建[cloud-consumer-feign-hystrix-order80](https://github.com/Silincee/springcloud2020/tree/main/cloud-consumer-feign-hystrix-order80)

2.POM

3.YML

4.主启动

5.业务类

- PaymentHystrixService
- OrderHystrixController

6.正常测试:http://localhost/consumer/payment/hystrix/ok/31

7.高并发测试

- 2W个线程压8001
- 消费端80微服务再去访问正常的OK微服务8001地址
- http://localhost/consumer/payment/hystrix/timeout/31
- **消费者80，要么转圈圈等待,要么消费端报超时错误**

![image-20201123210738488](/assets/imgs/image-20201123210738488.png)

### 故障现象和导致原因

8001同一层次的其他接口服务被困死，因为tomcat线程里面的工作线程已经被挤占完毕

80此时调用8001，客户端访问响应缓慢，转圈圈

### 结论

正因为有上述故障或不佳表现，才有我们的降级/容错/限流等技术诞生

## 如何解决？解决的要求

超时导致服务器变慢（转圈） 超时不再等待

出错（宕机或程序运行出错） 出错要有兜底

解决方法 🤔：

- **对方服务（8001）超时了，调用者（80）不能一直卡死等待，必须有服务降级**
- **对方服务（8001）宕机了，调用者（80）不能一直卡死等待，必须有服务降级**
- **对方服务（8001）OK，调用者（80）自己出故障或有自我要求（自己的等待时间小于服务提供者），自己处理降级**



## 服务降级

1.降低配置 `@HystrixCommand`

2.8001先从自身找问题

设置自身调用超时时间的峰值，峰值内可以正常运行，超过了需要有兜底的方法处理，作服务降级fallback

3.8001fallback

- **[业务类启用](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-hystrix-payment8001/src/main/java/cn/silince/springcloud/service/impl/PaymentServiceImpl.java)** @HystrixCommand报异常后如何处理
  - 一旦调用服务方法失败并抛出了错误信息后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法

- **[主启动类激活](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-hystrix-payment8001/src/main/java/cn/silince/springcloud/PaymentHystrixMain8001.java)** 添加新注解`@EnableCircuitBreaker`
- 当前服务不可用了，做服务降级，兜底的方案都是`paymentInfo_TimeOutHandler()`

```java
@Service
public class PaymentServiceImpl implements PaymentService {

    /**
     * @description: 正常访问，肯定ok
     */
    @Override
    public String paymentInfo_OK(Integer id) {
        return "线程池: "+ Thread.currentThread().getName()+"  paymentInfo_OK: "+id+"\t" + "😄";
    }

    // 设置自身调用超时时间的峰值，峰值内可以正常运行，超过了需要有兜底的方法处理，作服务降级fallback
    @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "3000") // 该线程超时时间为3s
    })
    @Override
    public String paymentInfo_TimeOut(Integer id) {
        int timeNumber = 5; // 5>3
//        int age = 10/0;
        try {
            TimeUnit.SECONDS.sleep(timeNumber);} catch (InterruptedException e) {e.printStackTrace();}
        return "线程池: "+ Thread.currentThread().getName()+"  paymentInfo_TimeOut: "+id+"\t" + "😭耗时(s): "+timeNumber;
    }

    public String paymentInfo_TimeOutHandler(Integer id) {
        return "线程池: "+ Thread.currentThread().getName()+"  系统繁忙,请稍后再试: "+id+"\t" + "😈Here is TimeOutHandler";
    }
}
```



4.80fallback(一般都放在客户端)

- 80订单微服务，也可以更好的保护自己，自己也依样画葫芦进行客户端降级保护
- ⚠️ **我们自己配置过的热部署方式对java代码的改动明显，但对`@HystrixCommand`内属性的修改建议重启微服务**
- [YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-hystrix-order80/src/main/resources/application.yml)  `feign.hystrix.enabled=true`
- [主启动](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-hystrix-order80/src/main/java/cn/silince/springcloud/OrderHystrixMain80.java) `@EnableHystrix`
- [业务类](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-hystrix-order80/src/main/java/cn/silince/springcloud/controller/OrderHystrixController.java)

```java
@RestController
@Slf4j
public class OrderHystrixController {

    @Autowired
    private PaymentHystrixService paymentHystrixService;

    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        return paymentHystrixService.paymentInfo_OK(id);
    }

    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "1500") // 该线程超时时间为1.5s,表示只等服务提供者1.5s
    })
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        return paymentHystrixService.paymentInfo_TimeOut(id);
    }

    public String paymentTimeOutFallbackMethod(Integer id) {
        return "😈Here is paymentTimeOutFallbackMethod, 我是消费者80，对方支付系统繁忙，请10秒钟后再试或者自己运行出错请检查自己";
    }

}
```

 5.目前问题

- 每个业务方法对应一个兜底的方法，代码膨胀

- 统一和自定义代码未分开

6.解决问题

***1）每个方法配置一个？？？膨胀***

- `@DefaultProperties(defaultFallback = "")`
- [controller配置](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-hystrix-order80/src/main/java/cn/silince/springcloud/controller/OrderHystrixController.java)

![image-20201123224105001](/assets/imgs/image-20201123224105001.png)

***2）和业务逻辑混一起？？？混乱***

- 服务降级，客户端去调用服务端，碰上服务端宕机或关闭
- **⭐️ 本次案例服务降级处理是在客户端80实现完成的，与服务端8001没有关系，只需要为Feign客户端定义的接口添加一个服务降级处理的实现类即可实现解耦**
- 未来我们要面对的异常 运行/超时/异常
- 再看我们的业务类`PaymentController`
- 修改`cloud-consumer-feign-hystrix-order80`
- 根据`cloud-consumer-feign-hystrix-order80`已经有的`PaymentHystrixService`接口，重新新建一个类**（`PaymentFallbackService`**）实现该接口，**统一为接口里面的方法进行异常处理**
- [PaymentFallbackService](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-hystrix-order80/src/main/java/cn/silince/springcloud/service/PaymentFallbackService.java)类实现`PaymentFeignClientService`接口
- [YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-hystrix-order80/src/main/resources/application.yml)

```yml
# 用于服务降级 在注解@FeignClient中添加fallbackFactory属性值
feign:
  hystrix:
    enabled: true # 在Feign中开启Hystrix
```

- [PaymentFeignClientService接口](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-hystrix-order80/src/main/java/cn/silince/springcloud/service/PaymentHystrixService.java)

```java
@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = PaymentFallbackService.class)
public interface PaymentHystrixService {

    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}
```

- 测试
  - 单个eureka先启动7001
  - PaymentHystrixMain8001启动
  - 正常访问测试 http://localhost/consumer/payment/hystrix/ok/31
  - 故意关闭微服务8001
  - 客户端自己调用提升；此时服务端provider已经down了，但是我们做了服务降级处理，让客户端在服务端不可用时也会获得提示信息而不会挂起耗死服务器



## 服务熔断

一句话就是家里保险丝。

熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。

**当检测到该节点微服务调用响应正常后，逐渐恢复调用链路。**

在Spring Cloud框架里，熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，

当失败的调用到一定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。熔断机制的注解是`@HystrixCommand`.





### 实操

1.修改[cloud-provider-hystrix-payment8001](https://github.com/Silincee/springcloud2020/tree/main/cloud-provider-hystrix-payment8001)

2.[PaymentService](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-hystrix-payment8001/src/main/java/cn/silince/springcloud/service/impl/PaymentServiceImpl.java)

涉及到断路器的三个重要参数:**快照时间窗、请求总数阀值、错误百分比阀值。**

- **快照时间窗**:断路器确定是否打开需要统计一些请求和错误数据， 而统计的时间范围就是快照时间窗，默认为最近的10秒。
- **请求总数阀值**:在快照时间窗内，必须满足请求总数阀值才有资格熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用次数不足20次即使所有的请求都超时或其他原因失败，断路器都不会打开。
- **错误百分比阀值**:当请求总数在快照时间窗内超过了阀值，比如发生了30次调用，如果在这30次调用中，有15次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%阀值情况下，这时候就会将断路器打开。

3.[PaymentController](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-hystrix-payment8001/src/main/java/cn/silince/springcloud/controller/PaymentController.java)

4.测试

- 自测cloud-provider-hystrix-payment8001
  - 正确 http://localhost:8001/payment/circuit/31
  - 错误 http://localhost:8001/payment/circuit/-31
- **重点测试：多次错误,然后慢慢正确，发现刚开始不满足条件，就算是正确的访问地址也不能进行访问，需要慢慢的恢复链路**



### 原理总结

熔断类型:

- 熔断打开

  请求不再进行调用当前服务，内部设置时钟一般为MTTR(平均故障处理时间)，当打开时长达到所设时钟则进入熔断状态

- 熔断关闭

  熔断关闭不会对服务进行熔断

- 熔断半开

  部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断

![image-20201124095515910](/assets/imgs/image-20201124095515910-6188057.png)

官网步骤：

![image-20201124100930639](/assets/imgs/image-20201124100930639-6188079.png)

断路器开启或者关闭的条件：

1. 当满足一定阀值的时候（默认10秒内超过20个请求次数）
2. 当失败率达到一定的时候（默认10秒内超过50%请求失败）
3. 到达以上阀值，断路器将会开启
4. 当开启的时候，所有请求都不会进行转发
5. 一段时间之后（默认是5秒），这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，若失败，继续开启。重复4和5

断路器打开之后:

1. 再有请求调用的时候，将不会调用主逻辑，而是直接调用降级fallback。通过断路器，实现了自动地发现错误并将降级逻辑切换为主逻辑，减少响应延迟的效果。

2. 原来的主逻辑要如何恢复呢？

   对于这一问题，hystrix也为我们实现了 自动恢复功能。

   当断路器打开，对主逻辑进行熔断之后， hystrix会启动一个休眠时间窗， 在这个时间窗内，熔断逻辑是临时的成为主逻辑。

   当休眠时间窗到期，断路器将进入半开状态，释放一次请求到原来的主逻辑上，如果此次请求正常返回，那么断路器将继续闭合。

   主逻辑恢复，如果这次请求依然有问题，断路器继续进入打开状态，休眠时间窗重新计时。

### ALL配置

![image-20201124113020614](/assets/imgs/image-20201124113020614.png)

![image-20201124113043907](/assets/imgs/image-20201124113043907.png)

![image-20201124113307490](/assets/imgs/image-20201124113307490.png)

![image-20201124113334648](/assets/imgs/image-20201124113334648.png)



## 服务限流

[见alibaba的Sentinel]()





# hystrix工作流程

> https://github.com/Netflix/Hystrix/wiki/How-it-Works

The following sections will explain this flow in greater detail:

1. [Construct a `HystrixCommand` or `HystrixObservableCommand` Object](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow1)
2. [Execute the Command](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow2)
3. [Is the Response Cached?](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow3)
4. [Is the Circuit Open?](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow4)
5. [Is the Thread Pool/Queue/Semaphore Full?](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow5)
6. [`HystrixObservableCommand.construct()` or `HystrixCommand.run()`](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow6)
7. [Calculate Circuit Health](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow7)
8. [Get the Fallback](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow8)
9. [Return the Successful Response](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow9)

![image-20201124113613545](/assets/imgs/image-20201124113613545.png)



# 服务监控hystrixDashboard

## 概述

除了隔离依赖服务的调用以外，Hytrix还提供了**准实时的调用监控（Hystrix Dashboard）**，Hystrix会持续地记录 所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请多少成功，多少失败等。Netflix通过

`hystrix-metrics-event-stream`项目实现了对以生指标的监控。Spring Cloud也提供了Hystrix Dashboard的整合，对监控内容转化成可视化界面。

## 仪表盘9001

1.新建[cloud-consumer-hystrix-dashboard9001](https://github.com/Silincee/springcloud2020/tree/main/cloud-consumer-hystrix-dashboard9001)

2.POM

3.YML

4.HystrixDashboardMain9001+**新注解@EnableHystrixDashboard**

5.**所有Provider微服务提供类（8001/8002/8003）都需要监控依赖配置**

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

6.启动cloud-consumer-hystrix-dashboard9001该微服务后续将监控微服务8001:http://localhost:9001/hystrix

## 断路器演示

1.修改cloud-provider-hystrix-payment8001

- 注意：新版本Hystrix需要在主启动类MainAppHystrix8001中指定监控路径，不然会报 `Unable to connect to Command Metric Stream`错误

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class PaymentHystrixMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class, args);

    }

    /**
     *此配置是为了服务监控而配置，与服务容错本身无关，springcloud升级后的坑
     *ServletRegistrationBean因为springboot的默认路径不是"/hystrix.stream"，
     *只要在自己的项目里配置上下面的servlet就可以了
     */
    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```



2.监控测试

1.9001监控8001

![image-20201124120437644](/assets/imgs/image-20201124120437644.png)

2.测试地址

- http://localhost:8001/payment/circuit/31

- http://localhost:8001/payment/circuit/-31

- 上述测试通过 ok

- 先访问正确地址，再多次访问错误地址，再正确地址，会发现图示断路器都是慢慢放开的

  - 监控结果，成功

    ![image-20201124120806458](/assets/imgs/image-20201124120806458.png)

  - 监控结果，失败

    ![image-20201124120834749](/assets/imgs/image-20201124120834749.png)

## 图例说明

- 7色

  ![image-20201124121109511](/assets/imgs/image-20201124121109511.png)

- 1圈

  实心圆:共有两种含义。它通过颜色的变化代表了实例的健康程度，它的健康度从`绿色<黄色<橙色<红色`递减。

  该实心圆除了颜色的变化之外，它的大小也会根据实例的请求流量发生变化，流量越大该实心圆就越大。所以通过该实心圆的展示，就可以在大量的实例中快速的发现**故障实例和高压力实例**。

- 1线

  曲线:用来记录2分钟内流量的相对变化，可以通过它来观察到流量的上升和下降趋势。

- 整图说明

  ![image-20201124121215757](/assets/imgs/image-20201124121215757.png)

- 整图说明2

![image-20201124121330510](/assets/imgs/image-20201124121330510.png)