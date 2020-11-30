---
layout: post
title: "SpringCloud Alibaba Seata处理分布式事务"
date: 2020-11-30 11:18:16 +0800--
categories: [Java, 分布式]
tags: [Java, 分布式, SpringCloud, Seata]  

---

# 分布式事务问题

分布式前：单机单库没这个问题，从1：1 ---> 1:N ---> N: N

单体应用被拆分成微服务应用，原来的三个模块被拆分成**三个独立的应用**，分别使用**三个独立的数据源**，业务操作需要调用三个服务来完成。此时**每个服务内部的数据一致性由本地事务来保证**，**但是全局的数据一 致性问题没法保证。**

***一句话：一次业务操作需要跨多个数据源或需要跨多个系统进行远程调用，就会产生分布式事务问题。***

![image-20201130100527634](/assets/imgs/image-20201130100527634.png)

# [Seata简介](http://seata.io/zh-cn/)

Seata是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。



## 一个典型的分布式事务过程

***一ID + 三组件模型：***

- 全局唯一的事务ID（Transaction ID---XID）
- 3组件概念
  - Transaction Coordinator(TC) ：事务协调者，事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚;
  - Transaction Manager(TM) ：事务管理器，控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议;
  - Resource Manager(RM) ：资源管理器，控制分支事务，负责分支注册，状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚；

***处理过程：***

1. TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID；
2. XID 在微服务调用链路的上下文中传播；
3. RM向TC注册分支事务，将其纳入XID对应全局事务的管辖；
4. TM向TC发起针对XID的全局提交或回滚决议； 
5. TC调度XID下管辖的全部分支事务完成提交或回滚请求。

![image-20201130102951096](/assets/imgs/image-20201130102951096.png)

## 如何使用

Spring的 本地@Transactional 控制事务注解

全局@GlobalTransactional	

***SEATA的分布式交易解决方案：***

![image-20201130125707248](/assets/imgs/image-20201130125707248.png)

# Seata-Server安装

1.[下载地址](https://github.com/seata/seata/releases/tag/v0.9.0)

2.下载版本 seata-server-0.9.0.tar.gz

3.解压到指定目录并修改conf目录下的file.conf配置文件

1）先备份原始file.conf文件

2）主要修改：自定义事务组名称+事务日志存储模式为db+数据库连接信息

- service模块`vgroup_mapping.my_test_tx_group = "fsp_tx_group"`
- store模块   ` mode = "file"`修改为`db`，并修改数据源信息。

```properties
mode = "db"
  url = "jdbc:mysql://127.0.0.1:3306/seata"
  user = "root"
  password = "你自己的密码"
```

3）mysql5.7数据库新建库seata

4）在seata库里建表；建表db_store.sql在`Seata/seata/conf`目录里面

5）修改`Seata/seata/conf`目录下的`registry.conf`配置文件：指明注册中为nacos，及修改nacos链接信息

![image-20201130132041342](/assets/imgs/image-20201130132041342.png)

6）**先启动Nacos**端口号8848，再启动seata-server

```shell
# Nacos
cd /Users/silince/Applications/Nacos/nacos/bin
sh startup.sh -m standalone
# seata-server
cd /Users/silince/Applications/Seata/seata/bin
sh seata-server.sh
```

# 订单/库存/账户业务数据库准备

## 分布式事务业务说明

下订单-->扣库存-->减账户（余额）

这里我们会创建三个服务， 一个订单服务，一个库存服务， 一个账户服务。

**当用户下单时，会在订单服务中创建一个订单， 然后通过远程调用库存服务来扣减下单商品的库存，再通过远程调用账户服务来扣减用户账户里面的余额，最后在订单服务中修改订单状态为已完成。**

该操作跨越三个数据库，有两次远程调用，很明显会有分布式事务问题。



## 创建业务数据库

seata_order: 存储订单的数据库

seata_storage:存储库存的数据库

seata_account: 存储账户信息的数据库

```sql
CREATE DATABASE seata_order;
CREATE DATABASE seata_storage;
CREATE DATABASE seata_account;
```

## 按照上述3库分别建对应业务表

1.seata_order库下建t_order表

```sql
CREATE TABLE t_order(
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
    `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
    `count` INT(11) DEFAULT NULL COMMENT '数量',
    `money` DECIMAL(11,0) DEFAULT NULL COMMENT '金额',
    `status` INT(1) DEFAULT NULL COMMENT '订单状态：0：创建中; 1：已完结'
) ENGINE=INNODB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;
 
SELECT * FROM t_order;
```

2.seata_storage库下建t_storage表

```sql
CREATE TABLE t_storage(
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
   `total` INT(11) DEFAULT NULL COMMENT '总库存',
    `used` INT(11) DEFAULT NULL COMMENT '已用库存',
    `residue` INT(11) DEFAULT NULL COMMENT '剩余库存'
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
 
INSERT INTO seata_storage.t_storage(`id`,`product_id`,`total`,`used`,`residue`)
VALUES('1','1','100','0','100');
 
SELECT * FROM t_storage;
```

3.seata_account库下建t_account表

```sql
CREATE TABLE t_account(
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY COMMENT 'id',
    `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
    `total` DECIMAL(10,0) DEFAULT NULL COMMENT '总额度',
    `used` DECIMAL(10,0) DEFAULT NULL COMMENT '已用余额',
    `residue` DECIMAL(10,0) DEFAULT '0' COMMENT '剩余可用额度'
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
 
INSERT INTO seata_account.t_account(`id`,`user_id`,`total`,`used`,`residue`) VALUES('1','1','1000','0','1000');
 
SELECT * FROM t_account;
```

4.按照上述3库分别建对应的回滚日志表

- 订单-库存-账户3个库下都需要建各自的回滚日志表
- `/Seata/seata/conf`目录下的 db_undo_log.sql 
- 建表SQL

```sql
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

# 订单/库存/账户业务微服务准备

## 业务需求

下订单->减库存->扣余额->改（订单）状态

## 新建订单Order-Module

1.[seata-order-service2001](https://github.com/Silincee/springcloud2020/tree/main/seata-order-service2001)

2.[POM](https://github.com/Silincee/springcloud2020/blob/main/seata-order-service2001/pom.xml)

```xml
<!--seata-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>seata-all</artifactId>
      <groupId>io.seata</groupId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>io.seata</groupId>
  <artifactId>seata-all</artifactId>
  <version>0.9.0</version>
</dependency>
```

3.[YML](https://github.com/Silincee/springcloud2020/blob/main/seata-order-service2001/src/main/resources/application.yml)

```yml
server:
  port: 2001

spring:
  application:
    name: seata-order-service
  cloud:
    alibaba:
      seata:
        #自定义事务组名称需要与seata-server中的对应
        tx-service-group: fsp_tx_group
    nacos:
      discovery:
        server-addr: localhost:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seata_order
    username: root
    password: 123456

feign:
  hystrix:
    enabled: false

logging:
  level:
    io:
      seata: info

mybatis:
  mapperLocations: classpath:mapper/*.xml
```

4.[file.conf ](https://github.com/Silincee/springcloud2020/blob/main/seata-order-service2001/src/main/resources/file.conf)

注意`vgroup_mapping.fsp_tx_group = "default" #修改自定义事务组名称`和db的设置

5.[registry.conf](https://github.com/Silincee/springcloud2020/blob/main/seata-order-service2001/src/main/resources/registry.conf)

```
  type = "nacos"

  nacos {
    serverAddr = "localhost:8848"
    namespace = ""
    cluster = "default"
  }
```

6.[domain](https://github.com/Silincee/springcloud2020/tree/main/seata-order-service2001/src/main/java/cn/silince/springcloud/alibaba/domain)

7.[Dao接口及实现](https://github.com/Silincee/springcloud2020/tree/main/seata-order-service2001/src/main/java/cn/silince/springcloud/alibaba/dao)

8.[Service接口及实现](https://github.com/Silincee/springcloud2020/tree/main/seata-order-service2001/src/main/java/cn/silince/springcloud/alibaba/service)

9.[Controller](https://github.com/Silincee/springcloud2020/tree/main/seata-order-service2001/src/main/java/cn/silince/springcloud/alibaba/controller)

10.[Config配置](https://github.com/Silincee/springcloud2020/tree/main/seata-order-service2001/src/main/java/cn/silince/springcloud/alibaba/config)

11.[主启动](https://github.com/Silincee/springcloud2020/blob/main/seata-order-service2001/src/main/java/cn/silince/springcloud/alibaba/SeataOrderMainApp2001.java)

取消数据源的自动创建，使用自己的数据源

`@SpringBootApplication(exclude = DataSourceAutoConfiguration.class) `



## 新建库存Storage-Module

1.[seata-storage-service2002](https://github.com/Silincee/springcloud2020/tree/main/seata-storage-service2002)

2.POM

3.YML

4.file.conf

5.registry.conf

6.domain

7.Dao接口及实现

8.Service接口及实现

9.Controller

10.Config配置

11.主启动



## 新建账户Account-Module

1.[seata-account-service2003](https://github.com/Silincee/springcloud2020/tree/main/seata-account-service2003)

2.POM

3.YML

4.file.conf

5.registry.conf

6.domain

7.Dao接口及实现

8.Service接口及实现

9.Controller

10.Config配置

11.主启动



## 测试

http://localhost:2001/order/create?userid=1&producrid=1&counr=10&money=100

case：

- 正常下单

- 超时异常(AccountServiceImpl添加超时)，没加@GlobalTransactional

  - 当库存和账户余额扣减后，订单状态并没有设置为已经完成，没有从零改为1
  - 而且由于feign的重试机制，账户余额还有可能被多次扣减

- 添加超时(AccountServiceImpl添加超时)，超时异常，添加@GlobalTransactional

  ![image-20201130212738567](/assets/imgs/image-20201130212738567.png)

  - 下单后数据库数据并没有任何改变;记录都添加不进来

  