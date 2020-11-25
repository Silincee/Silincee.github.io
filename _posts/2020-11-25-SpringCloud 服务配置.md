---
layout: post
title: "SpringCloud 服务配置"
date: 2020-11-25 14:11:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud]  

---

# SpringCloud config分布式配置中心

> 官网：https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/

## 概述

***1.分布式系统面临的配置问题：***

微服务意味着要将单体应用中的业务拆分成一个个子服务 ，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。

SpringCloud提供了ConfigServer来解决这个问题，我们每一 个微服务自己带着一个application.yml，上百个配置文件的管理。

***2.SpringCloud config 是什么：***

![image-20201125184309983](/assets/imgs/image-20201125184309983.png)

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，**配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。**

SpringCloud Config分为服务端和客户端两部分。

**服务端也称为分布式配置中心，它是一个独立的微服务应用，**用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。

客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。



***3.能干嘛：***

- 集中管理配置文件

- 不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release

- 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息

- 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置

- 将配置信息以REST接口的形式暴露：post、curl访问刷新均可。

  

***4.与Github整合配置***

由于SpringCloud Config默认使用Git来存储配置文件（也有其它方式，比如支持svn和本地文件，但最推荐的还是Git，而且使用的是http/https访问的形式）



## Config服务端配置与测试

1.用你自己的账号在Github上新建一个名为[sprincloud-config](https://github.com/Silincee/springcloud-config)的新Repository

2.本地硬盘上新建git仓库并clone

3.新建Module模块[cloud-config-center-3344](https://github.com/Silincee/springcloud2020/tree/main/cloud-config-center-3344)它既为Cloud的配置中心模块cloudConfig Center

- YML

```yml
server:
  port: 3344

spring:
  application:
    name:  cloud-config-center #注册进Eureka服务器的微服务名
  cloud:
    config:
      server:
        git:
          uri: https://github.com/Silincee/springcloud-config.git #GitHub上面的git仓库名字
          username: s****e
          password: ********
        ####搜索目录
          search-paths:
            - springcloud-config
      ####读取分支
      label: main
```

4.修改hosts文件，增加映射 `127.0.0.1 config-3344.com`

5.测试通过Config微服务是否可以从Github上获取配置内容

- 启动微服务3344
- http://config-3344.com:3344/main/config-dev.yml

6.配置读取规则

- `label：分支(branch)`  `name：服务名`  `profiles：环境(dev/test/prod)`
- `/{label}/{application}-{profile}.yml  `  推荐⭐️
  - main分支
    - http://config-3344.com:3344/main/config-dev.yml
    - http://config-3344.com:3344/main/config-test.yml
    - http://config-3344.com:3344/main/config-prod.yml
  - dev分支
    - http://config-3344.com:3344/dev/config-dev.yml
    - http://config-3344.com:3344/dev/config-test.yml
    - http://config-3344.com:3344/dev/config-prod.yml

- `/{application}-{profile}.yml`
- `/{application}-{profile}[/{label}]`



## Config客户端配置与测试

### 配置步骤

1.新建cloud-config-client-3355

- POM
- bootstrap.yml
- 主启动
- 业务类
- 测试：
  - http://localhost:3355/configInfo 
  - 成功实现了客户端3355访问SpringCloud Config3344通过GitHub获取配置信息



### 什么是bootstrap.yml

applicaiton.yml是用户级的资源配置项

bootstrap.yml是系统级的，**优先级更加高**

Spring Cloud会创建一个"Bootstrap Context”， 作为Spring应用的Application Context'的**父上下文**。初始化的时候，BootstrapContext'负责从**外部源**加载配置属性并解析配置。这两个上下文共享一个从外部获取Environment。

Bootstrap 属性有高优先级，默认情况下，它们不会被本地配置覆盖。`Bootstrap context`和`Application Context'有着不同的约定所以新增了一个bootstrap.ymI文件，保证`Bootstrap Context'和`Application Context`配置的分离。

**要将Client模块下的application.yml文件改为bootstrap.yml，这是很关键的，**因为bootstrap.yml是比application.yml先加载的。bootstrap.yml优先级高于application.yml





## Config客户端之动态刷新



### 分布式配置的动态刷新

1. Linux运维修改GitHub上的配置文件内容做调整
2. 刷新3344，发现ConfigServer配置中心立刻响应
3. 刷新3355，发现ConfigServer客户端没有任何响应
4. 3355没有变化除非自己重启或者重新加载
5. 难道每次运维修改配置文件，客户端都需要重启？？噩梦



### 动态刷新配置步骤

> 避免每次更新配置都要重启客户端微服务3355

1.修改3355模块

2.POM引入actuator监控

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

3.修改YML，暴露监控端口

```yml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

4.@RefreshScope业务类Controller修改

```java
@RestController
@RefreshScope
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

5.此时修改github---> 3344 ---> 3355

- ⚠️ 还是没有用！！！
- 需要运维人员发送Post请求刷新3355 ：curl -X POST "http://localhost:3355/actuator/refresh"
- 成功实现了客户端3355刷新到最新配置内容，避免了服务的重启

### 想想还有什么问题？

假如有多个微服务客户端3355/3366/3377...

每个微服务都要执行一次post请求，手动刷新？

**可否广播，一次通知，处处生效？**

我们想大范围的自动刷新，求方法

解决方法：[消息总线！！！](http://www.silince.cn/2020/11/25/SpringCloud-消息总线/)