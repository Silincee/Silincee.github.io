---
layout: post
title: "SpringCloud 服务降级Hystrix"
date: 2020-11-23 19:46:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# 分布式系统的服务雪崩

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。

![image-20201123195006080](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201123195006080.png)



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

![image-20201123204311947](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201123204311947.png)

![image-20201123204542479](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201123204542479.png)

![image-20201123204646299](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201123204646299.png)

![WX20201123-204845](/Users/silince/Develop/博客/blog_to_git/assets/imgs/WX20201123-204845.png)

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

![image-20201123210738488](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201123210738488.png)

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

![image-20201123224105001](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20201123224105001.png)

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

## 服务限流



# hystrix工作流程

# 服务监控hystrixDashboard

