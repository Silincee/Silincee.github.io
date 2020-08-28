---
layout: post
title:  "Redis集群"
date:   2019-12-30 14:20:06 +0800--
categories: [数据库]
tags: [redis, Java, ]  
---

## Redis集群

***容量不够，redis如何扩容？***

***并发写操作，redis如何分摊？***

### 什么是Redis集群？

***Redis集群实现了对Redis的水平扩容，即启动N个redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N。***

***Redis集群通过分区（partition）来提供一定程度的可用性（availability）***：即使集群中有一部分节点失效或者无法进行通讯，集群也可以继续处理命令请求。





### 节点

1. 一个集群至少要有三个主节点，即要有六个节点。

2. 分配原则尽量保证每个主数据库运行在不同的ip地址，每个从库和主库不在一个ip地址。

3. ***当主节点崩了，从节点能自动升为主节点；当主节点再次恢复时，主节点变为slave。参考哨兵模式。***

4. redis.conf有个参数cluster-require-full-coverage

   ```shell
   # 默认情况下，集群全部的slot有节点负责，集群状态才为ok，才能提供服务。设置为no，可以在slot没有全部分配的时候提供服务。不建议打开该配置。
   # cluster-require-full-coverage yes
   ```

   


### SLOTS

- 一个Redis 集群包含16384个插槽(hash slot)， 数据库中的每个键都属于这16384个插槽的其中一个，集群使用公式CRC1 6(key)% 16384来计算键key属于哪个槽(如果有组的话就只算组的部分)，其中`CRC16(key)`语句用于计算键key的CRC16校验和。

- 集群中的每个节点负责处理一部分插槽。 举个例子， 如果一个集群可以有主节点。其中:
  - 节点A负责处理0号至5500号插槽
  - 节点B负责处理5501号至11000号插槽
  - 节点C负责处理11001号至16383号插槽

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(注意：每个节点分配的插槽具体数字可能不同，当然可以通过一个小脚本来指定)



**一个疑问：为什么是16384(2^14)，而不是65535(2^16)呢？**

在redis节点发送心跳包时需要把所有的槽放到这个心跳包里，以便让节点知道当前集群信息，16384=16k，在发送心跳包时使用char进行bitmap压缩后是2kb（16384÷8÷1024=2kb），也就是说使用2k的空间创建了16k的槽数65535=65k，压缩后就是8kb（65536÷8÷1024=8kb），也就是说需要需要8k的心跳包。



### Redis Cluster原理

1. node1和node2首先进行握手meet，知道彼此的存在
2. 握手成功后，两个节点会定期发送ping/pong消息，交换数据信息(消息头，消息体)
3. 消息头里面有个字段：unsigned char myslots[CLUSTER_SLOTS/8]，每一位代表一个槽，如果该位是1，代表该槽属于这个节点
4. 消息体中会携带一定数量的其他节点的信息，大约占集群节点总数量的十分之一，至少是3个节点的信息。节点数量越多，消息体内容越大。
5. 每秒都在发送ping消息。每秒随机选取5个节点，找出最久没有通信的节点发送ping消息。
6. 每100毫秒都会扫描本地节点列表，如果发现节点最近一次接受pong消息的时间大于cluster-node-timeout/2,则立即发送ping消息

redis集群的主节点数量基本不可能超过1000个，超过的话可能会导致网络拥堵。



### 在集群中录入值(组的概念)

redis-cli客户端提供 **-c参数实现自动重定向**

```shell
redis-cli -c -p 6379
```

不在一个slot下的键值，是不能使用mget，mset等多键操作

可以通过{}来定义`组的概念`，从而使key中{}内相同内容的键值对放到一个slot中去。

```shell
set user:{info}:name xxx
set age{info} 12
set {info}email 12345@qq.com
hset user{info} name jiang
hset user{info} age 19
hset user{info} eamil 12345@qq.com

#结果
172.17.0.3:6379> keys *
1) "user{info}"
2) "{info}email"
3) "user:{info}:name"
4) "age{info}"
------------------------------------------------------
172.17.0.3:6379> hkeys user{info}
1) "name"
2) "age"
3) "eamil"
```



### 集群命令

```shell
CLUSTER INFO 打印集群的信息 
CLUSTER NODES 列出集群当前已知的所有节点（node），以及这些节点的相关信息。  

//节点(node) 
CLUSTER MEET <ip> <port> 将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。 
CLUSTER FORGET <node_id> 从集群中移除 node_id 指定的节点。 
CLUSTER REPLICATE <node_id> 将当前节点设置为 node_id 指定的节点的从节点。 
CLUSTER SAVECONFIG 将节点的配置文件保存到硬盘里面。  

//槽(slot) 
CLUSTER ADDSLOTS <slot> [slot ...] 将一个或多个槽（slot）指派（assign）给当前节点。 
CLUSTER DELSLOTS <slot> [slot ...] 移除一个或多个槽对当前节点的指派。 
CLUSTER FLUSHSLOTS 移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点。 
CLUSTER SETSLOT <slot> NODE <node_id> 将槽 slot 指派给 node_id 指定的节点，如果槽已经指派给另一个节点，那么先让另一个节点删除该槽>，然后再进行指派。 
CLUSTER SETSLOT <slot> MIGRATING <node_id> 将本节点的槽 slot 迁移到 node_id 指定的节点中。 
CLUSTER SETSLOT <slot> IMPORTING <node_id> 从 node_id 指定的节点中导入槽 slot 到本节点。 
CLUSTER SETSLOT <slot> STABLE 取消对槽 slot 的导入（import）或者迁移（migrate）。  

//键 (key) 
CLUSTER KEYSLOT <key> 计算键 key 应该被放置在哪个槽上。 
CLUSTER COUNTKEYSINSLOT <slot> 返回槽 slot 目前包含的键值对数量。 
CLUSTER GETKEYSINSLOT <slot> <count> 返回 count 个 slot 槽中的键。
```



### 集群搭建

https://blog.csdn.net/qq_42815754/article/details/82912130