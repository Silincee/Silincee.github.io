---
layout: post
title: "SpringCloud Alibaba Nacos服务注册和配置中心"
date: 2020-11-28 19:18:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# SpringCloud Alibaba入门简介

## 为什么会出现SpringCloud alibaba

[Spring Cloud Netflix项目进入维护模式](https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now)

***什么是维护模式:***

将模块置于维护模式，意味着Spring Cloud团队将不会再向模块添加新功能。我们将修复block级别的bug以及安全问题，我们也会考虑并审查社区的小型pull request.

***进入维护模式意味着什么呢？***

Spring Cloud Netlix将不再开发新的组件 

我们都知道Spring Cloud版本迭代算是比较快的，因而出现了很多重大ISSUE都还来不及Fix就又推另一个Release了.进入维护模式意思就是目前一直以后 一段时间Spring Cloud Netflix提供的服务和功能就这么多了，不在开发新的组件和功能了。以后将以维护和Merge分支Full Request为主。

**新组件功能将以其他替代平代替的方式实现**



## SpringCloud alibaba是什么

***诞生:***

2018.10.31， [Spring Cloud Alibaba](https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md)正式入驻了Spring Cloud官方孵器，并在Maven中央库发布了第一个版本。

Spring Cloud for Alibaba，它是由一些阿里巴巴的开源组件和云产品组成的。 这个项目的目的是为了让大家所熟知的Spring框架，其优秀的设计模式和抽象理念，以给使用阿里巴巴产品的Java开发者带来使用Spring Boot和Spring Cloud的更多便利。

***功能：***

- 服务限流降级:默认支持Servlet. Feign. RestTemplate、 Dubbo 和RocketMQ限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级Metrics监控。
- 服务注册与发现:适配Spring Cloud服务注册与发现标准，默认集成了Ribbon的支持。
- 分布式配置管理:支持分布式系统中的外部化配置，配置更改时自动刷新。
- 消息驱动能力:基于Spring Cloud Stream为微服务应用构建消息驱动能力。
- 阿里云对象存储:阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- 分布式任务调度:提供秒级、精准、可靠、高可用的定时（基于Cron表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子务均均分配到所有Worker （schedulerx-client）上执行。

***怎么玩：***

![image-20201127201408218](/assets/imgs/image-20201127201408218.png)

***学习资料：***

- 官网：https://spring.io/projects/spring-cloud-alibaba#overview
- 英文：https://github.com/alibaba/spring-cloud-alibaba / https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html
- 中文：https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md



# Nacos服务注册和配置中心

> [中文文档](https://nacos.io/zh-cn/index.html)			 [英文文档](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_discovery)			[下载地址](https://github.com/alibaba/nacos/releases/tag/1.1.4)

## Nacos简介

为什么叫Nacos：前四个字母分别为Naming和Configuration的前两个字母，最后的s为Service。

Nacos是一个更易于构建云原生应用的动态服务发现，配置管理和服务管理中心。Nacos就是注册中心+配置中心的组合，等价于 `Eureka+Config+Bus`。可以替代Eureka做服务注册中心，替代Config做服务配置中心。

各种注册中心比较:

![image-20201127205627362](/assets/imgs/image-20201127205627362.png)

据说Nacos在阿里巴巴内部有超过10万的实例运行，已经过了类似双十一等各种大型流量的考验。



## 安装并运行Nacos

- 解压安装包，直接运行bin目录下的`startup.sh`
- 命令运行成功后直接访问http://localhost:8848/nacos/#/login
- 默认账号密码都是nacos

```shell
cd /Users/silince/Applications/Nacos/nacos/bin
sh startup.sh -m standalone
```

![image-20201127211652219](/assets/imgs/image-20201127211652219.png)

# Nacos作为服务注册中心演示

## 基于Nacos的服务提供者

1.新建Module [cloudalibaba-provider-payment9001](https://github.com/Silincee/springcloud2020/tree/main/cloudalibaba-provider-payment9001)

2.POM

- 父POM

  ```xml
  <!--spring cloud alibaba 2.1.0.RELEASE-->
  <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.1.0.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
  </dependency>
  ```

- 本模块POM

  ```xml
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
  ```

3.[YML](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-provider-payment9001/src/main/resources/application.yml)

```yml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

4.[主启动](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-provider-payment9001/src/main/java/cn/silince/springcloud/PaymentMain9001.java)

5.[业务类](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-provider-payment9001/src/main/java/cn/silince/springcloud/controller/PaymentController.java)

6.测试

- http://localhost:9001/payment/nacos/1

- nacos控制台

- 为了下一章节演示nacos的负载均衡，参照9001新建`cloudalibaba-provider-payment9002`

  ![image-20201127214141788](/assets/imgs/image-20201127214141788.png)



## 基于Nacos的服务消费者

1.新建Module [cloudalibaba-consumer-nacos-order83](https://github.com/Silincee/springcloud2020/tree/main/cloudalibaba-consumer-nacos-order83)

2.[POM](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order83/pom.xml)

3.[YML](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order83/src/main/resources/application.yml)

4.[主启动](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order83/src/main/java/cn/silince/springcloud/alibaba/OrderNacosMain83.java)

5.业务类

- [ApplicationContextBean](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order83/src/main/java/cn/silince/springcloud/alibaba/config/ApplicationContextConfig.java)	`@LoadBalance`
- [OrderNacosController](https://github.com/Silincee/springcloud2020/blob/main/cloudalibaba-consumer-nacos-order83/src/main/java/cn/silince/springcloud/alibaba/controller/OrderNacosController.java)

6.测试

- nacos控制台
- http://localhost:83/consumer/payment/nacos/13  83访问9001/9002，轮询负载OK





## 服务注册中心对比

![image-20201127222043334](/assets/imgs/image-20201127222043334.png)

### Nacos和CAP

![image-20201127221922015](/assets/imgs/image-20201127221922015.png)

### AP和CP模式的切换

**C是所有节点在同一 时间看到的数据是一 致的；而A的定义是所有的请求都会收到响应。**

何时选择使用何种模式？

- **如果不需要存储服务级别的信息且服务实例是通过`nacos-client`注册， 并能够保持心跳上报，那么就可以选择AP模式。**当前主流的服务如Spring cloud和Dubbo服务，都适用于AP模式，AP模式为了服务的可用性而减弱了一致性， 因此AP模式下只支持注册临时实例。
- **如果需要在服务级别编辑或者存储配置信息，那么CP是必须**，K8S服务和DNS服务则适用于CP模式。
- **CP模式下则支持注册持久化实例**，此时则是以Raft协议为集群运行模式，**该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。**

`curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'`



# Nacos作为服务配置中心演示

## Nacos作为配置中心-基础配置

1.新建 [cloudalibaba-config-nacos-client3377]()

2.POM

```xml
<!--nacos-config-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<!--nacos-discovery-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

3.[application.yml]()	[bootstrap.yml]()

Nacos同`springcloud-config`一样，在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动。

springboot中配置文件的加载是存在优先级顺序的，bootstrap优先级高于application

4.[主启动]()

5.[业务类]()

6.在Nacos中添加配置信息

***[Nacos中的匹配规则:](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)***

![image-20201128193308367](/assets/imgs/image-20201128193308367.png)

***实操:***

1）配置新增

2）Nacos界面配置对应

- 公式 `${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}`
- prefix默认为spring.application.name的值
- spring.profile.active既为当前环境对应的profile,可以通过配置项spring.profile.active 来配置
- file-exetension为配置内容的数据格式，可以通过配置项spring.cloud.nacos.config.file-extension配置

![image-20201128193856203](/assets/imgs/image-20201128193856203.png)

3）总结

![image-20201128194034336](/assets/imgs/image-20201128194034336.png)

---

7.测试

- 启动前需要在nacos客户端-配置管理-配置管理栏目下有没有对应的yaml配置文件
- 运行cloud-config-nacos-client3377的主启动类
- 调用接口查看配置信息 http://localhost:3377/config/info
- 自带动态刷新



## Nacos作为配置中心-分类配置

1.存在的问题

***问题1:***

实际开发中，通常一个系统会准备dev开发环境/test测试环境/prod生产环境。如何保证指定环境启动时服务能正确读取到Nacos上相应环境的配置文件呢？

***问题2:*** 

一个大型分布式微服务 系统会有很多微服务子项目，每个微服务项又都会有相应的开发环境、测试环境、预发环境、正式环境....那怎么对这些微服务配置进行管理呢？

2.Nacos的图形化管理界面

***配置管理：***

![image-20201128195748218](/assets/imgs/image-20201128195748218.png)

***命名空间：***

![image-20201128195709287](/assets/imgs/image-20201128195709287.png)

---

3.`Namespace+Group+Data ID`三者关系？为什么这么设计？

类似Java里面的package名和类名。

最外层的namespace是可以用于区分部署环境的，Group和DatalD逻辑上区分两个目标对象。

⭐️ 默认情况: 

- Namespace = public
- Group = DEFAULT_GROUP
- 默认 Cluster 为 ADEFAULT

![image-20201128195907641](/assets/imgs/image-20201128195907641.png)

Nacos默认的命名空间是public， Namespace主要用来实现隔离。

比方说我们现在有三个环境:开发、测试、生产环境，我们就可以创建三个Namespace，不同的Namespace之间是隔离的。

Group默认是DEFAULT_GROUP， Group可以把不同的微服务划分到同一个分组里面去

Service就是微服务； 一个Service可以包含多个Cluster（集群），Nacos默认Cluster是DEFAULT， Cluster是对指定微服务的一个虚拟划分。

比方说为了容灾，将Service微服务分别部署在了杭州机房和广州机房，

这时就可以给杭州机房的Service微服务起一个集群名称（HZ） ，

给广州机房的Service微服务起一个集群名称（GZ），还可以尽量让同一个机房的微服务互相调用，以提升性能。

最后是Instance，就是微服务的实例。

---

4.案例

***1）DataID方案：***

- 指定spring.profile.active和配置文件的DataID来使不同环境下读取不同的配置
- 默认空间+默认分组+新建dev和test两个DataID
- 通过`spring.profile.active`属性就能进行多环境下配置文件的读取

***2）Group方案***

- 通过Group实现环境区分 新建Group`DEV/TEST_GROUP`和配置文件DataID `nacos-config-client-info.yaml`

![image-20201128202712296](/assets/imgs/image-20201128202712296.png)

- 在config下增加一条group的配置即可。可配置为DEV_GROUP或TEST_GROUP

![image-20201128202813902](/assets/imgs/image-20201128202813902.png)

***3）Namespace方案***

- 新建dev/test的Namespace

![image-20201128203500307](/assets/imgs/image-20201128203500307.png)

- 回到服务管理-服务列表查看

![image-20201128204221748](/assets/imgs/image-20201128204221748.png)

- 按照域名配置填写application和bootstrap

```yml
# bootstrap.yml
spring.cloud.nacos.config.group: DEV/TEST_GROUP
spring.cloud.nacos.config.namespace: f3d2e9a7-d5f7-4c33-9624-f5c0a2bfb798/...
# application.yml
spring.profiles.active: dev/test # 表示开发环境
```





# Nacos集群和持久化配置 🤔

## [官网说明](https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html)

***集群部署架构图：***

因此开源的时候推荐用户把所有服务列表放到一个vip（虚拟ip，即nginx）下面，然后挂到一个域名下面

[http://ip1](http://ip1/):port/openAPI 直连ip模式，机器挂则需要修改ip才可以使用。

[http://VIP](http://vip/):port/openAPI 挂载VIP模式，直连vip即可，下面挂server真实ip，可读性不好。

[http://nacos.com](http://nacos.com/):port/openAPI 域名 + VIP模式，可读性好，而且换ip方便，推荐模式 ⭐️

![image-20201128205955119](/assets/imgs/image-20201128205955119.png)

上图的翻译，真实情况如下：

![image-20201128210335683](/assets/imgs/image-20201128210335683.png)

## [Nacos支持三种部署模式](https://nacos.io/zh-cn/docs/deployment.html)

- 单机模式 - 用于测试和单机试用。
- 集群模式 - 用于生产环境，确保高可用。
- 多集群模式 - 用于多数据中心场景。

**默认Nacos使用嵌入式数据库实现数据的存储。**所以，如果启动多个默认配置下的Nacos节点，数据存储**会出现一致性问题**的。为了解决这个问题，***Nacos采用了集中式存储的方式来支持集群化部署，目前只支持MySQL的存储***



## Nacos持久化配置解释

Nacos默认自带的是嵌入式数据库[derby](https://github.com/alibaba/nacos/blob/develop/config/pom.xml)

derby到mysql切换配置步骤:

1）nacos-server-1.1.4\nacos\conf目录下找到sql脚本。赋值去mysql中执行

2）nacos-server-1.1.4\nacos\conf目录下找到application.properties

- 修改conf/application.properties文件，增加支持mysql数据源配置（目前只支持mysql），添加mysql数据源的url、用户名和密码。

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://11.162.196.16:3306/nacos_devtest?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=nacos_devtest
db.password=youdontknow
```

3）重新启动nacos，可以看到是个全新的空记录界面，以前是记录进derby



## Linux版Nacos+MySQL生产环境配置

1.Linux服务器上mysql数据库配置，具体步骤上方已说明

2.Linux服务器上nacos的集群配置cluster.conf

- 梳理出3台nacos机器的不同服务端口号
- 复制出cluster.conf。 

```shell
cd /root/nacos/nacos/conf
cp cluster.conf.example cluster.conf
vim cluster.conf
# 添加以下内容 ⚠️ 这个IP不能写127.0.0.1,必须是Linux命令hostname -i能够识别的IP
47.97.191.157:3333
47.97.191.157:4444
47.97.191.157:5555
```

3.编辑Nacos的启动脚本startup.sh，使它能够接受不同的启动端口

- /mynacos/nacos/bin目录下有startup.sh
- 备份startup.sh后修，修改内容：
  - `nohup $JAVA -Dserver.port=${PORT} ${JAVA_OPT} nacos.nacos`

![image-20201128221813954](/assets/imgs/image-20201128221813954.png)

![image-20201128221827100](/assets/imgs/image-20201128221827100.png)

- 执行方式 `./startup.sh -p 3333`

4.Nginx的配置，由它作为负载均衡器

- 修改nginx的配置文件 nginx.conf

  ![image-20201128223608341](/assets/imgs/image-20201128223608341.png)

- 按照指定启动 `./nginx -c /usr/local/nginx/conf/nginx.conf`

5.截止到此处，1个Nginx+3个nacos注册中心+1个mysql

- 启动nacos  `./startup.sh -p 3333/4444/5555`  ⚠️ 内存不足可能启动不了3台
- 查看集群启动状态 `ps -ef|grep nacos|grep -v grep|wc -l`

- 测试通过nginx访问nacos  `https://写你自己虚拟机的ip:1111/nacos/#/login`

- linux服务器的mysql插入一条记录  ok

6.测试

- 微服务cloudalibaba-provider-payment9002启动注册进nacos集群

  ```yml
  # 修改配置文件为集群模式
  server:
    port: 9002
  
  spring:
    application:
      name: nacos-payment-provider
    cloud:
      nacos:
        discovery:
          # 换成nginx的1111端口，做集群
          server-addr: 192.168.111.144:1111
  
  management:
    endpoints:
      web:
        exposure:
          include: '*'
  ```

7.总结

![image-20201128225947836](/assets/imgs/image-20201128225947836.png)



### 注意事项

1）内存不足可能启动不了3台

2）发布失败。请检查参数是否正确

原因：Nacos1.1.4版本更换Mysql版本为8.0.16版本以上出现的问题。nacos的jdbc版本太低，需要改动源码，和你的mysql版本一致，修改POM文件

```xml
<!-- JDBC libs -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.16</version>
</dependency>
```

