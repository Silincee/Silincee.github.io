---
layout: post
title: "SpringCloud 服务调用"
date: 2020-11-22 22:01:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# Ribbon负载均衡服务调用

> 官网资料：https://github.com/Netflix/ribbon/wiki/Getting-Started
>
> Ribbon目前也进入维护模式

## 概述

Spring Cloud Ribbon是基于Netfli Ribbon实现的一套**客户端负载均衡的工具。**

简单的说，Ribbon是Netflix发 布的开源项目，主要功能是提供**客户端的软件负载均衡算法和服务调用**。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer （简称LB） 后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。



能干嘛：

- LB（负载均衡）
  - 集中式LB
  - 进程内LB
- 前面我们讲解过了80通过轮询负载访问8001/8002
- 一句话：负载均衡+RestTemplate调用



## LB负载均衡（Load Balance）

简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA （高可用）。

常见的负载均衡有软件Nginx， LVS， 硬件F5等。

**🤔 Ribbon本地负载均衡客户端VS Nginx服务端负载均衡区别**

Nginx是服务器负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即**负载均衡是由服务端实现的。**

**Ribbon本地负载均衡**，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在**本地实现RPC远程服务调用技术。**

### 集中式LB 

即在服务的消费方和提供方之间使用独立的LB设施（可以是硬件，如F5， 也可以是软件，如nginx）， **由该设施负责把访问请求通过某种策略转发到服务的提供方；**

### 进程内LB

将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。**Ribbon就属于进程内LB，它只是一个类库， 集成于消费方进程，消费方通过它来获取到服务提供方的地址。**



## Ribbon负载均衡演示

1.架构说明

**总结：Ribbon其实就是一个软负载均衡的客户端组件，他可以和其他所需请求的客户端结合使用，和eureka结合只是其中的一个实例。**

![image-20201123094412190](/assets/imgs/image-20201123094412190.png)

Ribbon在工作时分成两步：

- 第一步先选择 EurekaServer,它优先选择在同一个区域内负载较少的 server.
- 第二步再根据用户指定的策略,在从 server取到的服务注册列表中选择一个地址。
- 其中 Ribbon提供了多种策略:比如轮询、随机和根据响应时间加权。



2.POM

 `spring-cloud-starter-netflix-eureka-client`自带了Ribbon



3.RestTemplate的使用

- [官网](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html)
- getForObject方法/getForEntity方法

```java
// 返回对象为响应体中数据转化成的对象，基本可以理解为Json
@GetMapping("/consumer/payment/get/{id}")
public CommonResult<Payment> getPayment(@PathVariable("id")Long  id){
  return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);
}

// 返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体
@GetMapping("/consumer/payment/getForEntity/{id}")
public CommonResult<Payment> getPayment2(@PathVariable("id")Long  id){
  ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
  if (entity.getStatusCode().is2xxSuccessful()){
    return entity.getBody();
  }else {
    return new CommonResult(444,"操作失败");
  }
}
```

- postForObject/postForEntity

```java
@GetMapping("/consumer/payment/create")
public CommonResult<Payment> create(Payment payment){
  return restTemplate.postForObject(PAYMENT_URL+"/payment/create",payment, CommonResult.class);
  // return restTemplate.postForEntity(PAYMENT_URL+"/payment/create",payment, CommonResult.class).getBody();
}
```





## Ribbon核心组件IRule

**1.IRule:根据特定算法中从服务列表中选取一个要访问的服务 🤔**

- com.netflix.loadbalancer.RoundRobinRule 
  - 轮询
- com.netflix.loadbalancer.RandomRule
  - 随机
- com.netflix.loadbalancer.RetryRule
  - 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试
- WeightedResponseTimeRule 
  - 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
- BestAvailableRule 
  - 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
- AvailabilityFilteringRule 
  - 先过滤掉故障实例，再选择并发较小的实例
- ZoneAvoidanceRule
  - 默认规则，复合判断server所在区域的性能和server的可用性选择服务器

![image-20201123102100659](/assets/imgs/image-20201123102100659-6098324.png)



2.如何替换

- [修改cloud-consumer-order80](https://github.com/Silincee/springcloud2020/tree/main/cloud-consumer-order80)
- ***注意配置细节***
  - 官方文档明确给出了警告:这个自定义配置类不能放在`@ComponentScan`所扫描的当前包下以及子包下，**否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的了。**
  - 主启动类的`@SpringBootApplication`下包含了`@ComponentScan`,因此配置类不能放在`cn.silince.springcloud`包及其子包下
- 新建package:`cn.silince.myrule`
- 上面包下新建MySelfRule规则类

```java
public class MySelfRule {
    
    @Bean
    public IRule myRule(){
        return new RandomRule(); // 定义为随机
    }
}
```

- 主启动类添加@RibbonClient
  - `@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)`
- 测试：http://localhost/consumer/payment/get/31





## Ribbon负载均衡算法

### 原理

**负载均衡算法: rest接口第几次请求数%服务器集群总数量=实际调用服务器位置下标，每次服务重启动后rest接口计数从1开始。**

`List<Servicelnstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");`

如:	 List [0] instances = 127.0.0.1:8002；List [1] instances = 127.0.0.1:8001

8001+ 8002组合成为集群，它们共计2台机器，集群总数为2，按照轮询算法原理:

- 当总请求数为1时: 1 %2 =1对应下标位置为1，则获得服务地址为127.0.0.1:8001
- 当总请求数位2时: 2 % 2 =0对应下标位置为0，则获得服务地址为127.0.0.1:8002
- 当总请求数位3时: 3 %2 =1对应下标位置为1，则获得服务地址为127.0.0.1:8001
- 当总请求数位4时: 4 %2 =0对应下标位置为0，则获得服务地址为127.0.0.1:8002
- 如此类推.....



### RoundRobinRule源码

```java
// 原子整型类
private AtomicInteger nextServerCyclicCounter;

public Server choose(ILoadBalancer lb, Object key) {
  if (lb == null) {
    log.warn("no load balancer");
    return null;
  }

  Server server = null;
  int count = 0;
  while (server == null && count++ < 10) {
    List<Server> reachableServers = lb.getReachableServers(); // 活着/可达健康的机器列表
    List<Server> allServers = lb.getAllServers(); //  所有服务器的列表
    int upCount = reachableServers.size();
    int serverCount = allServers.size(); // 服务器集群总数量

    if ((upCount == 0) || (serverCount == 0)) { // 无可用机器
      log.warn("No up servers available from load balancer: " + lb);
      return null;
    }
		
    int nextServerIndex = incrementAndGetModulo(serverCount); //得到余数
    server = allServers.get(nextServerIndex); // 根据取模后的余数选择机器

    if (server == null) {
      /* Transient. */
      Thread.yield(); // 使当前线程由执行状态，变成为就绪状态
      continue;
    }

    if (server.isAlive() && (server.isReadyToServe())) {
      return (server);
    }

    // Next.
    server = null;
  }

  if (count >= 10) {
    log.warn("No available alive servers after 10 tries from load balancer: "
             + lb);
  }
  return server;
}
/**
 * Inspired by the implementation of {@link AtomicInteger#incrementAndGet()}.
 *
 * @param modulo The modulo to bound the value of the counter.
 * @return The next value.
 */
private int incrementAndGetModulo(int modulo) { // modulo:服务器集群总数量
  // 自旋锁
  for (;;) {
    int current = nextServerCyclicCounter.get();
    int next = (current + 1) % modulo; // 取余操作
    if (nextServerCyclicCounter.compareAndSet(current, next)) // CAS比较并设值
      return next; // 返回余数
  }
}
```





### 自定义负载均衡算法

1.7001/7002集群启动

2.8001/8002微服务controller改造，添加以测试方法

```java
/** 
 * @description: 自定义负载均衡算法 
 */ 
@GetMapping(value = "/payment/lb")
public String getPaymentLB(){
  return serverPort;
}
```

3.80订单微服务改造

- ApplicationContextBean去掉`@LoadBalanced`
- [LoadBalancer接口](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-order80/src/main/java/cn/silince/springcloud/lb/LoadBalancer.java)

```java
/**
* @description: 自定义负载均衡算法接口
*/
public interface LoadBalancer {

    /**
    * @description: 获取所有服务实例列表
    */
    ServiceInstance instances(List<ServiceInstance> serviceInstances);
    
}
```

- [MyLB](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-order80/src/main/java/cn/silince/springcloud/lb/MyLb.java)

```java
/**
 * @program: cloud2020
 * @description: 自定义负载均衡算法实现类
 * @author: Silince
 * @create: 2020-11-23 13:16
 **/
@Component // 使得容器能扫描得到该类
public class MyLb implements LoadBalancer {

    private AtomicInteger atomicInteger = new AtomicInteger(0); // 原子整型类

    public final int getAndIncrement() {
        int current;
        int next;

        do {
            current = this.atomicInteger.get();
            next = current >= Integer.MAX_VALUE ? 0 : current + 1;
        } while (!this.atomicInteger.compareAndSet(current,next)); // CAS自旋
        System.out.println("****第几次访问，次数next: "+next);
        return next;
    }


    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstances) {
        int index =  getAndIncrement() % serviceInstances.size();
        return serviceInstances.get(index);
    }
}
```

- [OrderController](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-order80/src/main/java/cn/silince/springcloud/controller/OrderController.java)

```java
@RestController
@Slf4j
public class OrderController {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private LoadBalancer loadBalancer;

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/consumer/payment/lb")
    public String getPaymentLB(){
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        if (instances==null||instances.size()<=0){
            return null;
        }
        ServiceInstance serviceInstance = loadBalancer.instances(instances);
        URI uri = serviceInstance.getUri();
        return restTemplate.getForObject(uri+"/payment/lb",String.class);
    }
}
```

- 测试:http://localhost/consumer/payment/lb



# OpenFeign服务接口调用

## 概述

1.[OpenFeign](https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/#spring-cloud-openfeign)是什么

GitHub:https://github.com/spring-cloud/spring-cloud-openfeign

**Feign是一个声明式的web服务客户端，让编写web服务客户端变得非常容易，只需创建一个接口并在接口上添加注解即可。** ***一句话，Feign就是一个服务接口绑定器。***

它的使用方法是**定义一个服务接口然后在上面添加注解**。Feign也支持可拔插式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和HttpMessageConverters。**Feign可以与Eureka和Ribbon组合使用以支持负载均衡。**



2.能干嘛

***Feign旨在使编写Java Http客户端变得更容易。***

前面在使用Ribbon+ RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，**往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一 些客户端类来包装这些依赖服务的调用。**所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下,**我们只需创建一个接口并使用注解的方式来配置它（以前是Dao接口 上面标注Mapper注解，现在是一个微服务接口 上面标注一个Feign注解即可）**， 即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时， 自动封装服务调用客户端的开发量。

***Feign集成了Ribbon***

利用Ribbon维护了Payment的服务列表信息，并通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，**通过feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用**。



3.Feign和OpenFeign两者区别

![image-20201123180627382](/assets/imgs/image-20201123180627382.png)



## 使用步骤

1.微服务调用接口+@FeignClient

2.新建[cloud-consumer-feign-order80](https://github.com/Silincee/springcloud2020/tree/main/cloud-consumer-feign-order80) ，Feign在消费端使用

3.[POM](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-order80/pom.xml)

4.[YML](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-order80/src/main/resources/application.yml)

5.[主启动类]()

6.业务类

- [新建PaymentFeignService接口并新增注解@FeignClient](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-order80/src/main/java/cn/silince/springcloud/service/PaymentFeignService.java)
- [控制层Controller](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-order80/src/main/java/cn/silince/springcloud/controller/OrderFeignController.java)

7.测试

- 先启动2个eureka集群7001/7002
- 再启动2个微服务8001/8002
- 启动OpenFeign启动
- http://localhost/consumer/payment/get/31
- Feign自带负载均衡配置项

### 总结

![image-20201123190409506](/assets/imgs/image-20201123190409506.png)

Feign与ribbon+restTemplate不同，可以使程序员使用更为熟悉的面向接口编程。

![image-20201123190857514](/assets/imgs/image-20201123190857514.png)





## 超时控制

1.超时设置，故意设置超时演示出错情况

- [服务提供方8001故意写暂停程序](https://github.com/Silincee/springcloud2020/blob/main/cloud-provider-payment8001/src/main/java/cn/silince/springcloud/controller/PaymentController.java)

- [服务消费方80添加超时方法PaymentFeignService](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-order80/src/main/java/cn/silince/springcloud/service/PaymentFeignService.java)

- [服务消费方80添加超时方法OrderFeignController](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-order80/src/main/java/cn/silince/springcloud/controller/OrderFeignController.java)

- 测试

  - http://localhost/consumer/payment/feign/timeout
  - 错误页面

  ![image-20201123193246594](/assets/imgs/image-202011223193246594.png)



2.OpenFeign默认等待一秒钟，超过后报错

默认Feign客户端只等待秒钟， 但是服务端处理需要超过1秒钟，导致Feign客户端不想等待了，直接返回报错。为了避免这样的情况，有时候我们需要设置Feign客户端的超时控制。



3.OpenFeign默认支持Ribbon，YML文件里需要开启OpenFeign客户端超时控制

```yml
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
#指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
#指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```



## 日志打印功能

Feign提供了日志打功能，我们可以通过配置来调整日志级别，从而了解Feign中Http请求的细节。说白了就是**对Feign接口的调用情况进行监控和输出。**

日志级别

- NONE:默认的，不显示任何日志；
- BASIC:仅记录请求方法、URL、 响应状态码及执行时间；
- HEADERS:除了BASIC 中定义的信息之外，还有请求和响应的头信息；
- FULL:除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据。

使用步骤：

1.[配置日志bean](https://github.com/Silincee/springcloud2020/blob/main/cloud-consumer-feign-order80/src/main/java/cn/silince/springcloud/config/FeignConfig.java)

```java
@Configuration
public class FeignConfig {
    
    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}
```

2.YML文件里需要开启日志的Feign客户端

```yml
logging:
  level:
    # feign日志以什么级别监控哪个接口
    cn.silince.springcloud.service.PaymentFeignService: debug
```



3.后台日志查看

![image-20201123194438210](/assets/imgs/image-20201123194438210.png)