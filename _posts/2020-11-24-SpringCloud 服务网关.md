---
layout: post
title: "SpringCloud 服务网关"
date: 2020-11-24 12:11:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# Zuul

> 官网：https://github.com/Netflix/zuul/wiki

因为内部已出现了巨大分歧，已不推荐使用。 停止更新进入维护

Zuul2多次跳票。 研发中



# Gateway新一代网关

> 官网：https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/

## 概述简介

⭐️ ***一句话：Spring Cloud Gateway 使用的Webflux中的`reactor-netty`响应式编程组件，底层使用了Netty通讯框架。***

GateWay能干嘛：

- 反向代理
- 鉴权
- 流量控制
- 熔断
- 日志监控

Cloud全家桶中有个很重要的组件就是网关，在1.x版本中都是采用的Zuu网关；但在2.x版本中，zuu的升级直跳票， SpringCloud最后自己研发了一个网关替代Zuul，那就是SpringCloud Gateway一句话: **gateway是原zuul1 .x版的替代**

Gateway是在Spring生态系统之上构建的API网关服务，基基Spring 5， Spring Boot 2和Project Reactor等技术。

**Gateway旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能， 例如: 熔断、限流、重试等。**

SpringCloud Gateway是Spring Cloud的一个全新项目，基于Spring 5.0+ Spring Boot 2.0和Project Reactor等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的API路由管理方式。

SpringCloud Gateway作为Spring Cloud生态系统中的网关，目标是替代Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 1.x非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，***而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。***

Spring Cloud Gateway的目标提供统一的路由方式且基于Filter链的方式提供了网关基本的功能，例如:安全，监控/指标，和限流。



### 为什么选择GateWay

1.neflix不太靠谱，zuul2.0一直跳票,迟迟不发布

一方面因为Zuul1.0已经进入了维护阶段，且Gateway是SpringCloud团队研发的，亲儿子产品，值得信赖。且很多功能Zuul都没有用起来也非常的简单便捷。

**Gateway是基于异步非阻塞模型上进行开发的，性能方面不需要担心。**虽然Netflix早就发布 了最新的Zuul 2.x，但Spring Cloud貌似没有整合计划。而且Netflix相关组件都宣布进入维护期；不知前景如何？

多方面综合考虑Gateway是很理想的网关选择。



2.SpringCloud Gateway具有如下特性

- **基于Spring Framework 5， Project Reactor和Spring Boot 2.0进行构建；**
- 动态路由:能够匹配任何请求属性；
- 可以对路由指定Predicate （断言）和Filter （过滤器） ；
- 集成Hystrix的断路器功能； 
- 集成Spring Cloud服务发现功能；
- 易于编写的Predicate （断言）和Filter （过滤器） ；
- 请求限流功能；
- 支持路径重写。



3.SpringCloud Gateway与Zuul的区别

在 SpringCloud Finchley正式版之前, Spring Cloud推荐的网关是 Netflix提供的zuul:

- zuul1.x,是一个基于阻塞 l/O 的 API Gateway
- **Zuul1.x基于 Servlet 2.5使用阻塞架构**它不支持任何长连接(如WebSocket)Zuul设计模式和 Nginx较像,每次操作都是从工作线程中选择一个执行,请求线程被阻塞到工作线程完成,但是差别是 Nginx用C++实现,zuul用 Java实现,而 JVM 本身会有第一次加载较慢的情况，使得Zuul的性能相对较差。
- **Zuul2.x理念更先进,想基于 Netty非阻塞和支持长连接**,但 SpringCloud目前还没有整合zuul2.x的性能较Zuul 1.x有较大提升。在性能方面,根据官方提供的基准测试, Spring Cloud Gateway的RPS(每秒请求数)是Zuul的1.6倍。
- Spring Cloud Gateway建立在 Spring Framework5、 Project Reactor和 Spring Boot2之上,**使用非阻塞APl**
- Spring Cloud Gateway还支持 WebSocket,并且与 Spring紧密集成拥有更好的开发体验



### Zuul1.x模型和GateWay模型架构

#### Zuul1.x模型

Springcloud中所集成的Zuul版本，采用的是Tomcat容器，使用的是传统的Servlet IO处理模型。

**servlet由servlet container进行生命周期管理:**

- container启动时构造servlet对象并调用`servlet init()`进行初始化；
- container运行时接受请求，并为每个请求分配一个线程 （一般从线程池中获取空闲线程）然后调用`service()`。
- container关闭时调用`servlet destory()`销毁servlet；

![image-20201124135548961](/assets/imgs/image-20201124135548961.png)

上述模式的缺点:

servlet是一 个简单的网络10模型， 当请求进入servlet container时， servlet container就会为其绑定一个线程，**在并发不高的场景下这种模型是适用的。**但是一旦高并发（此如抽风用jemeter压测），线程数量就会上涨，而线程资源代价是昂贵的（上线文切换，内存消耗大）严重影响请求的处理时间。***在一些简单业务场景下，不希望为每个request分配一个线程，只需要1个或几个线程就能应对极大并发的请求，这种业务场景下servlet模型没有优势。***

**所以Zuul 1.X是基于servlet之上的一个阻塞式处理模型**，即spring实现了处理所有request请求的一个servlet （DispatcherServlet） 并由该servlet阻塞式处理处理。所以Springcloud Zuu|无法摆脱servlet模型的弊端



#### GateWay模型

WebFlux是什么？

传统的Web框架，比如说: struts2， springmvc等都是基于Servlet API与Servlet容器基础之上运行的。

但是**在Servlet3.1之后有了异步非阻塞的支持**。而`WebFlux`是一个典型非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty， Undertow及支持Servlet3.1的容器上。**非阻塞式+函数式编程（Spring5必须让你使用java8）。**

Spring WebFlux是Spring 5.0引入的新的响应式框架，区别于Spring MVC，它不需要依赖Servlet API，它是完全异步非阻塞的，并且基于Reactor来实现响应式流规范。



## 微服务架构中网关在哪里

![image-20201124140706447](/assets/imgs/image-20201124140706447.png)



## 三大核心概念

### Route(路由)

路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

### Predicate（断言）

参考的是java8的`java.util.function.Predicate`开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由

### Filter(过滤)

指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

### 总体

web请求，通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行些精细化控制。

predicate就是我们的匹配条件；而filter， 就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标uri，就可以实现一个具体的路由了

![image-20201124141014905](/assets/imgs/image-20201124141014905.png)



## Gateway工作流程

> **核心逻辑：路由转发+执行过滤器链**

客户端向Spring Cloud Gateway发出请求。然后在Gateway Handler Mapping中找到与请求相匹配的路由，将其发送到GatewayWeb Handler。

Handler再通过指定的**过滤器链**来将请求发送到我们实际的服务执行业务逻辑，然后返回。

过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前`（ "pre" ）`或之后`（ "post" ）`执行业务逻辑。

- **Filter在"pre" 类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、 协议转换等，**
- **在"post" 类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。**

![image-20201124141137845](/assets/imgs/image-20201124141137845.png)





## 入门配置

### 配置步骤

1.新建Module [cloud-gateway-gateway9527](https://github.com/Silincee/springcloud2020/tree/main/cloud-gateway-gateway9527)

2.POM

3.YML

4.主启动类

5.`9527网关`如何做路由映射 ?

- **我们目前不想暴露8001端口，希望在8001外面套一层9527**
- cloud-provider-payment8001看看controller的访问地址

6.YML新增网关配置

```yml
spring:
  application:
    name: cloud-gateway
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
```

7.测试

- 启动9527网关/cloud-provider-payment8001/7001/7002
- 添加网关前  http://localhost:8001/payment/get/31
- 添加网关后  http://localhost:9527/payment/get/31

![image-20201124155155399](/assets/imgs/image-20201124155155399.png)



### YML配置说明

Gateway网关路由有两种配置方式

1. 在配置文件yml中配置（上面已详细说明）

2. 代码中注入RouteLocator的Bean

   - 官网案例

     ![image-20201124191338000](/assets/imgs/image-20201124191338000.png)

   - 需求通过9527网关访问到外网的[百度新闻网址](http://news.baidu.com/guoji)

     ```java
     package cn.silince.springcloud.config;
     
     
     @Configuration
     public class GateWayConfig {
     
         /**
          * 配置了一个id为path_route_silince的路由规则，
          * 当访问地址 http://localhost:9527/guonei 时会自动转发到地址 
          * http://news.baidu.com/guonei
          * */
         @Bean
         public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder){
             RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
             routes.route("path_route_silince",
                     r->r.path("/guonei")
                             .uri("http://news.baidu.com/guonei")).build();
             return routes.build();
         }
     }
     ```

     





## 通过微服务名实现动态路由

默认情况下Gateway会根据注册中心的服务列表，**以注册中心上微服务名为路径创建动态路由进行转发**，从而实现**动态路由**的功能

修改 YML 配置文件：

`lb://` 表示从注册中心获取服务

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
```

测试：

- 启动一个eureka7001+两个服务提供者8001/8002
- 访问http://localhost:9527/payment/lb  
- 效果：8001/8002两个端口切换



## Predicate的使用

启动 `gateway9527 `时候的日志：

![image-20201124194121337](/assets/imgs/image-20201124194121337.png)



***Route Predicate Factories是什么:***

Spring Cloud Gateway将路由匹配作为Spring WebFlux HandlerMapping基础架构的一部分。

Spring Cloud Gateway包括许多内置的Route Predicate工厂。所有这些Predicate都与HTTP请求的不同属性匹配。多个RoutePredicate.厂可以进行组合

Spring Cloud Gateway创建Route对象时，使用RoutePredicateFactory创建Predicate对象，Predicate 对象可以赋值给Route。Spring Cloud Gateway包含许多内置的Route Predicate Factories。

所有这些谓词都匹配HTTP请求的不同属性。多种谓词工厂可以组合，并通过逻辑and。



***常用的Route Predicate:***

1.After Route Predicate 

-  在该时间之后才能访问，时间串获得方式`ZonedDateTime.now()`
- `- After=2020-11-24T19:49:17.786+08:00[Asia/Shanghai]`  

2.Before Route Predicate

- 在该时间之前才能访问
- `- Before=2020-11-24T19:49:17.786+08:00[Asia/Shanghai]`  

3.Between Route Predicate

- 在该时间段内才能访问
- `- Between=2020-11-24T19:49:17.786+08:00[Asia/Shanghai],2020-11-24T19:59:17.786+08:00[Asia/Shanghai]`

4.Cookie Route Predicate

- Cookie Route Predicate需要两个参数，一个是Cookie name ，一个是正则表达式。路由规则会通过获取对应的Cookie name值和正则表达式去匹配，如果匹配上就会执行路由，如果没有匹配上则不执行。
- `- Cookie=username,silince`
- ![image-20201124195724545](/assets/imgs/image-20201124195724545.png)
- ![image-20201124195902750](/assets/imgs/image-20201124195902750.png)

5.Header Route Predicate

- 两个参数:一个是属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行。
- `- Header=X-Request-Id, \d+`  请求头中要有X-Request-Id属性并且值为整数的正则表达式
- 测试：`curl http://localhost:9527/payment/lb --H "X-Request-Id:123"`

6.Host Route Predicate

- Host Route Predicate接收一组参数，一组匹配的域名列表，这个模板是一个`ant`分隔的模板，用`.`号作为分隔符。它通过参数中的主机地址作为匹配规则。
- `- Host=**.silince.cn`
- 测试：`curl http://localhost:9527/payment/lb --H "Host:www.silince.cn"`

7.Method Route Predicate

- `- Method=GET`   Get请求才允许路由

8.Path Route Predicate

- 路径匹配才能访问

9.Query Route Predicate

- `- Query=username, \d+ ` 要有参数名称并且是正整数才能路由

***总结：***

说白了，Predicate就是为了实现一组匹配规则，让请求过来找到对应的Route进行处理。



## Filter的使用

### Spring Cloud Gateway的Filter 

路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用。

Spring Cloud Gateway内置了多种路由过滤器，他们都由GatewayFilter的工厂产生。

生命周期：

- pre 业务逻辑之前
- post 业务逻辑之后

种类：

- [GatewayFilter 单一的](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#the-addrequestparameter-gatewayfilter-factory)
- [GlobalFilter 全局的](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#global-filters)

![image-20201124222202837](/assets/imgs/image-20201124222202837.png)

### 自定义过滤器

[自定义全局GlobalFilter](https://github.com/Silincee/springcloud2020/blob/main/cloud-gateway-gateway9527/src/main/java/cn/silince/springcloud/filter/MyLogGateWayFilter.java)

1.`impiemerts  GlobalFilter,Ordered`

2.能干嘛

- 全局日志记录
- 统一网关鉴权
- ...

3.案例代码

```java
@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("***come in MyLogGateWayFilter"+ new Date());
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname==null){
            log.info("***用户名为null，非法用户😈");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    /**
    * @description: 加载过滤器的顺序，数字越小优先级越高
    */
    @Override
    public int getOrder() {
        return 0;
    }
}
```

4.测试

- http://localhost:9527/payment/lb?uname=1 ok
- http://localhost:9527/payment/lb?name=1 被拦截