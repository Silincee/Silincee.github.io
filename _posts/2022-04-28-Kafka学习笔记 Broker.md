---
layout: post
title:  "Kafka学习笔记 Broker"
date:   2022-04-28 13:19:06 +0800--
categories: [Kafka]
tags: [Kafka, ]  
---

#  Kafka Broker

## Zk 存储结构

![image-20220428181857947](/assets/imgs/image-20220428181857947.png)

## Broker工作流程

![image-20220428182003980](/assets/imgs/image-20220428182003980.png)

## Kafka 副本

1）副本基本信息

- Kafka 副本作用:提高数据可靠性。
- Kafka 默认副本 1 个，生产环境一般配置为 2 个，保证数据可靠性;太多副本会增加磁盘存储空间，增加网络上数据传输，降低效率。
- Kafka 中副本分为:**Leader 和 Follower**。Kafka 生产者只会把数据发往 Leader， 然后 Follower 会找 Leader 进行同步数据。
- Kafka 分区中的所有副本统称为 AR(Assigned Repllicas)。 
  - AR = ISR + OSR
  - **ISR**，表示和 Leader 保持同步的 Follower 集合。如果 Follower 长时间未向 Leader 发送通信请求或同步数据，则该 Follower 将被踢出 ISR。该时间阈值由 **replica.lag.time.max.ms** 参数设定，默认 30s。Leader 发生故障之后，就会从 ISR 中选举新的 Leader。
  - **OSR**，表示 Follower 与 Leader 副本同步时，延迟过多的副本。





2）**Leader** 选举流程

Kafka 集群中有一个 broker 的 Controller 会被选举为 Controller Leader，负责管理集群 broker 的上下线，所有 topic 的分区副本分配和 Leader 选举等工作。Controller 的信息同步工作是依赖于 Zookeeper 的。

![image-20220428200354155](/assets/imgs/image-20220428200354155.png)



3）**Follower** 故障处理细节

- **LEO(Log End Offset)**:每个副本的最后一个offset，LEO其实就是最新的offset + 1。

- **HW(High Watermark)**:所有副本中最小的LEO 。

<img src="/assets/imgs/image-20220428221819121.png" alt="image-20220428201213159" style="zoom:50%;" />

此时follower2发生故障，Follower会读取本地磁盘记录的 上次的HW，并将1og文件高于HW的部分截取掉，从HW开始 向Leaderi进行同步。

![image-20220428201213159](/assets/imgs/image-20220428201213159.png)



4）**Leader** 故障处理细节

![image-20220428201236334](/assets/imgs/image-20220428201236334.png)

为保证多个副本之间的数据一致性，其余的Follower会先 将各自的log文件高于HW的部分截掉，然后从新的Leaderl同步数据。

<img src="/assets/imgs/image-20220428222212617.png" alt="image-20220428222212617" style="zoom:50%;" />

5）分区副本分配

> 如果 kafka 服务器只有 4 个节点，那么设置 kafka 的分区数大于服务器台数，在 kafka 底层如何分配存储副本呢?

创建 16 分区，3 个副本场景下会如下分配(示意图中省略了12-15分区)，目的是为了尽可能的保持==负载均衡==

![image-20220503232233272](/assets/imgs/image-20220503232233272.png)

![image-20220428223318373](/assets/imgs/image-20220428223318373.png)



## 生产经验

### 手动调整分区副本存储

> 需求🤔:创建一个新的topic，4个分区，两个副本，名称为three。将该topic的所有副本都存储到broker0和 broker1两台服务器上。

在生产环境中，每台服务器的配置和性能不一致，但是Kafka只会根据自己的代码规则创建对应的分区副本，就会导致个别服务器存储压力较大。所有需要手动调整分区副本的存储。

![image-20220503232436895](/assets/imgs/image-20220503232436895.png)

手动调整分区副本存储的步骤如下:

```shell
# 1.创建一个新的 topic，名称为 three。
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --partitions 4 --replication-factor 2 -- topic three
# 2.查看分区副本存储情况。
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic three
# 3.创建副本存储计划(所有副本都指定存储在 broker0、broker1 中)。
vim increase-replication-factor.json
{
   "version":1,
   "partitions":[{"topic":"three","partition":0,"replicas":[0,1]}, 
                 {"topic":"three","partition":1,"replicas":[0,1]}, 
                 {"topic":"three","partition":2,"replicas":[1,0]}, 
                 {"topic":"three","partition":3,"replicas":[1,0]}]
}
# 4.执行副本存储计划。
bin/kafka-reassign-partitions.sh -- bootstrap-server localhost:9092 --reassignment-json-file increase-replication-factor.json --execute
# 5.验证副本存储计划。
bin/kafka-reassign-partitions.sh -- bootstrap-server localhost:9092 --reassignment-json-file increase-replication-factor.json --verify
# 6.查看分区副本存储情况。
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic three
```



### **Leader Partition** 负载平衡

正常情况下，Kafka本身会自动把Leader Partition均匀分散在各个机器上，来保证每台机器的读写吞吐量都是均匀的。但是如果某些broker宕机，**会导致Leader Partition过于集中在其他少部分几台broker上**，这会导致少数几台broker的读写请求压力过高，其他宕机的 broker重启之后都是follower partition，读写请求很低，**造成集群负载不均衡**。

![image-20220503232919857](/assets/imgs/image-20220503232919857.png)

| 参数名称                                | 描述                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| auto.leader.rebalance.enable            | 默认是 true。 自动 Leader Partition 平衡。生产环 境中，==leader 重选举的代价比较大，可能会带来性能影响，建议设置为 false 关闭。== |
| leader.imbalance.per.broker.percentage  | 默认是 10%。每个 broker 允许的不平衡的 leader 的比率。如果每个 broker 超过了这个值，控制器会触发 leader 的平衡。 |
| leader.imbalance.check.interval.seconds | 默认值 300 秒。检查 leader 负载是否平衡的间隔 时间。         |



### 增加副本因子

在生产环境当中，由于某个主题的重要等级需要提升，我们考虑增加副本。副本数的增加需要先制定计划，然后根据计划执行。

```shell
# 1.创建 topic
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --partitions 3 --replication-factor 1 -- topic four
# 2.手动增加副本存储。创建副本存储计划(所有副本都指定存储在 broker0、broker1、broker2 中)。
vim increase-replication-factor.json
{"version":1,"partitions":[{"topic":"four","partition":0,"replicas":[0,1,2]},{"topic":"four","partition":1,"replicas":[0,1,2]},{"t opic":"four","partition":2,"replicas":[0,1,2]}]}
# 3.执行副本存储计划。
bin/kafka-reassign-partitions.sh -- bootstrap-server localhost:9092 --reassignment-json-file increase-replication-factor.json --execute
```







## 文件存储

### 文件存储机制

Topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是Producer生产的数据。

- Producer生产的数据会被不断==追加==到该log文件末端，为防止log文件过大导致数据定位效率低下，Kafka采取了分片和索引机制， 将每个partition分为多个segment。
- 每个segment包括:“.index”文件、“.log”文件和.timeindex等文件。这些文件位于一个文件夹下，该文件夹的命名规则为:topic名称+分区序号，例如:first-0。

![image-20220504232359653](/assets/imgs/image-20220504232359653.png)



**index** 文件和 **log** 文件详解(稀疏索引)：

- 相对偏移量：如 `522(起始offset)+65(相对offerset) = 587(绝对offset)`

![image-20220504232716835](/assets/imgs/image-20220504232716835.png)

日志存储参数配置：

| 参数                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| log.segment.bytes        | Kafka 中 log 日志是分成一块块存储的，此配置是指 log 日志划分 成块的大小，默认值 1G。 |
| log.index.interval.bytes | 默认 4kb，kafka 里面每当写入了 4kb 大小的日志(.log)，然后就 往 index 文件里面记录一个索引。 稀疏索引。 |

### 文件清理策略

Kafka 中默认的日志保存时间为 7 天，可以通过调整如下参数修改保存时间。 

- log.retention.hours，最低优先级小时，默认7天。
- log.retention.minutes，分钟。
-  log.retention.ms，最高优先级毫秒。
- log.retention.check.interval.ms，负责设置检查周期，默认5分钟。 

> 🤔那么日志留存时间一旦超过了设置的时间，怎么处理呢?

Kafka 中提供的日志清理策略有 `delete` 和 `compact` 两种。

1）**delete 日志删除:将过期数据删除**

- log.cleanup.policy = delete 所有数据启用删除策略
  - 基于时间:默认打开。以 segment 中所有记录中的最大时间戳作为该文件时间戳。 
  - 基于大小:默认关闭。超过设置的所有日志总大小，删除最早的 segment。log.retention.bytes，默认等于-1，表示无穷大。

> 🤔如果一个 segment 中有一部分数据过期，一部分没有过期，怎么处理?

如采用基于时间的删除方式，因为segment 中所有记录中的最大时间戳作为该文件时间戳，因此会将整个segment删除。如果是基于大小的删除方式则不影响。

![image-20220505160824746](/assets/imgs/image-20220505160824746.png)



2）**compact日志压缩**：对于相同key的不同value值，只保留最后一个版本。

- log.cleanup.policy = compact 所有数据启用压缩策略

![image-20220505161201918](/assets/imgs/image-20220505161201918.png)

压缩后的offset可能是不连续的，比如上图中没有6，当从这些offset消费消息时，将会拿到比这个offset大 的offset对应的消息，实际上会拿到offset为7的消息，并从这个位置开始消费。

⚠️==这种策略只适合特殊场景，比如消息的key是用户ID，value是用户的资料，通过这种压缩策略，整个消息集里就保存了所有用户最新的资料。==



## 高效读写

> [Kafka 是如何实现高吞吐率的](http://www.silince.cn/2021/08/27/Kafka-%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E9%AB%98%E5%90%9E%E5%90%90%E7%8E%87%E7%9A%84/#%E9%A1%BA%E5%BA%8F%E8%AF%BB%E5%86%99)

**1**)**Kafka** 本身是分布式集群，可以采用分区技术，并行度高 

**2**)读数据采用稀疏索引，可以快速定位要消费的数据 

**3**)顺序写磁盘

Kafka 的 producer 生产数据，要写入到 log 文件中，写的过程是一直追加到文件末端， 为顺序写。官网有数据表明，同样的磁盘，顺序写能到 600M/s，而随机写只有 100K/s。这 与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。

![image-20220505161340139](/assets/imgs/image-20220505161340139.png)

**4**)页缓存 **+** 零拷贝技术

- **零拷贝**:Kafka的数据加工处理操作交由Kafka生产者和Kafka消费者处理。Kafka Broker应用层不关心存储的数据，所以就不用走应用层，传输效率高。

- **PageCache页缓存**:Kafka重度依赖底层操作系统提供的PageCache功能。当上层有写操作时，操作系统只是将数据写入 PageCache。当读操作发生时，先从PageCache中查找，如果找不到，再去磁盘中读取。实际上PageCache是把尽可能多的空闲内存 都当做了磁盘缓存来使用。

  ![image-20220505161603741](/assets/imgs/image-20220505161603741.png)

相关参数：

- log.flush.interval.messages：强制页缓存刷写到磁盘的条数，默认是 long 的最大值， 9223372036854775807。一般不建议修改，交给系统自己管 理。
- log.flush.interval.ms：每隔多久，刷数据到磁盘，默认是 null。一般不建议修改， 交给系统自己管理。


