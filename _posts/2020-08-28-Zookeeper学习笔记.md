---
layout: post
title:  "Zookeeper学习笔记"
date:   2020-08-28 20:20:06 +0800--
categories: [Java]
tags: [Zookeeper, Java, ]  

---

# ZooKeeper学习笔记

## Zookeeper入门

### 概述

***Zookeeper是一个开源的分布式的，为分布式应用提供协调服务的 Apache 项目。***


Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，***它负责存储和管理大家都关心的数据，然后接受观察者的注册***，一旦这些数据的状态发生变化，Zookeeper就将***负责通知已经在Zookeeper上注册的那些观察者做出相应的反应。***

![](/assets/imgs/01.png)

1. 服务端启动时去注册信息（创建都是临时节点）
2. 获取到当前在线服务器列表，并且注册监听
3. 服务器节点下线
4. 服务器节点上下线事件通知
5. 重新再去获取服务器列表，并注册监听

一言蔽之：**ZooKeeper = 文件系统 + 通知机制**

### 特点

![](/assets/imgs/02.png)

1. Zookeeper：一个领导者（Leader），多个跟随者（Follower）组成的集群。
2. ***集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。***
3. ***全局数据一致***：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
4. 更新请求顺序进行，来自同一个Client的更新请求按其发送顺序依次执行。
5. ***数据更新原子性，一次数据更新要么成功，要么失败。***
6. ***实时性，在一定时间范围内，Client能读到最新数据。***

### 数据结构

ZooKeeper数据模型的结构与Unix文件系统很类似，***整体上可以看作是一棵树， 每个节点称做一个ZNode。每一个ZNode默认能够存储1MB的数据， 每个ZNode都可以通过其路径唯一标识。***

![](/assets/imgs/03.png)

### 应用场景

提供的服务包括：

1. 统一命名服务
2. 统一配置管理
3. 统一集群管理
4. 服务器节点动态上下线
5. 软负载均衡
6. ...

#### 统一命名服务 ####

在分布式环境下，经常需要对应用/服务进行统一命名，便于识别。

例如：IP不容易记住，而域名容易记住。

![](/assets/imgs/04.png)

#### 统一配置管理 ####

1. ***分布式环境下，配置文件同步非常常见***
	1. 一般要求一个集群中，所有节点的配置信息是一致的，比如Kafka集群
	2. 对配置文件修改后，希望能够快速同步到各个节点上
2. ***配置管理可交由ZooKeeper实现***
	1. 可将配置信息写入ZooKeeper上的一个Znode
	2. 各个客户端服务器监听这个Znode
	3. 一旦Znode中的数据被修改， ZooKeeper将通知各个客户端服务器

![](/assets/imgs/05.png)

#### 统一集群管理 ####

1. ***分布式环境中，实时掌握每个节点的状态是必要的***
	1. ***可根据节点实时状态做出一些调整***
2. ***ZooKeeper可以实现实时监控节点状态变化***
	1. ***可将节点信息写入ZooKeeper上的一个ZNode***
	2. ***监听这个ZNode可获取它的实时状态变化***

![](/assets/imgs/06.png)

#### 服务器动态上下线 ####

***客户端能实时洞察到服务器上下线的变化。***

![](/assets/imgs/01.png)

#### 软负载均衡 ####

***在Zookeeper中记录每台服务器的访问数， 让访问数最少的服务器去处理最新的客户端请求。***

![](/assets/imgs/07.png)

## Zookeeper安装

### 下载地址

在[下载地址](https://zookeeper.apache.org/releases.html)下载zookeeper-3.4.14。

### 本地模式安装 主要用于测试

#### Linux下安装,

##### 安装前准备 #####

1)安装JDK

2)将压缩包解压缩到`/Users/silince/Applications/ZooKeeper/`路径下

```shell
tar -zxvf zookeeper-3.4.14.tar.gz
```

##### 配置修改 #####

```shell
# 修改zoo_sample.cfg为zoo.cfg
cd /Users/silince/Applications/ZooKeeper/zookeeper-3.4.14/conf
mv zoo_sample.cfg zoo.cfg
# 在zoo.cfg中修改dataDir路径
dataDir=/Users/silince/Applications/ZooKeeper/zookeeper-3.4.14/zkData
# 在目录下创建zkData文件夹
mkdir zkData
```

##### 操作 Zookeeper #####

1）启动Zookeeper 

```
bin/zkServer.sh start
```

2）查看进程是否启动

```
jps
```

3）查看状态

```
bin/zkServer.sh status
```

4）启动客户端

```
bin/zkCli.sh
```

5）退出客户端

```
quit
```

6）停止Zookeeper

```
bin.zkServer.sh stop
```

### 配置参数解读

Zookeeper中的配置文件zoo.cfg中参数含义解读如下：

1. tickTime=2000： **通信心跳数， Zookeeper 服务器与客户端心跳时间，单位毫秒**<br>
Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。<br>
它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。 (session的最小超时时间是2*tickTime)
2. initLimit=10： **LF 初始通信时限**<br>
集群中的Follower跟随者服务器与Leader领导者服务器之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。
3. syncLimit=5： **LF 同步通信时限**<br>
集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit \* tickTime， Leader认为Follwer死掉，从服务器列表中删除Follwer。
4. dataDir：**数据文件目录+数据持久化路径**<br>
主要用于保存 Zookeeper 中的数据。
5. clientPort =2181：**客户端连接端口**<br>
监听客户端连接的端口。

## Zookeeper内部原理

### 选举机制

**面试重点**

1. ***半数机制：集群中半数以上机器存活，集群可用。所以 Zookeeper 适合安装奇数台服务器。例如，5台服务器有3台存活，集群可用，而只有2台存活，集群不可用。***
2. Zookeeper 虽然在配置文件中***并没有指定 Master 和 Slave***。 但是， Zookeeper 工作时，是有一个节点为 Leader，其他则为 Follower， Leader 是通过***内部的选举机制临时产生***的。
3. 以一个简单的例子来说明整个选举的过程。

假设有五台服务器组成的 Zookeeper 集群，它们的 id 从 1-5，同时它们都是最新启动的，也就是没有历史数据，在存放数据量这一点上，都是一样的。假设这些服务器依序启动，来看看会发生什么，如下图所示。

![](/assets/imgs/08.png)

1. 服务器 1 启动， 发起一次选举。服务器 1 投自己一票。此时服务器 1 票数一票，不够半数以上（3 票），选举无法完成，服务器 1 状态保持为 LOOKING；
2. 服务器 2 启动， 再发起一次选举。服务器 1 和 2 分别投自己一票并交换选票信息：此时服务器 1 发现服务器 2 的 ID 比自己目前投票推举的（服务器 1）大，更改选票为推举服务器 2。此时服务器 1 票数 0 票，服务器 2 票数 2 票，***没有半数以上结果，选举无法完成***，服务器 1， 2 状态保持 LOOKING
3. 服务器 3 启动， 发起一次选举。此时服务器 1 和 2 都会更改选票为服务器 3。此次投票结果：服务器 1 为 0 票，服务器 2 为 0 票，服务器 3 为 3 票。***此时服务器 3 的票数已经超过半数，服务器 3 当选 Leader***。服务器 1， 2 更改状态为 FOLLOWING，服务器 3 更改状态为 LEADING；
4. 服务器 4 启动， 发起一次选举。此时服务器 1， 2， 3 已经不是 LOOKING 状态，不会更改选票信息。交换选票信息结果：服务器 3 为 3 票，服务器 4 为 1 票。此时服务器 4服从多数，更改选票信息为服务器 3，并更改状态为 FOLLOWING；
5. 服务器 5 启动，同 4 一样当小弟。


### 节点类型

- ***持久（Persistent）：客户端和服务器端断开连接后， 创建的节点不删除***
- ***短暂（Ephemeral）：客户端和服务器端断开连接后， 创建的节点自己删除***

![](/assets/imgs/09.png)

1. **持久化目录节点** 

   客户端与Zookeeper断开连接后，该节点依旧存在

2. **持久化顺序编号目录节点** 

   客户端与Zookeeper断开连接后， 该节点依旧存在， 只是Zookeeper给该节点名称进行顺序编号

3. **临时目录节点** 

   客户端与Zookeeper断开连接后， 该节点被删除

4. **临时顺序编号目录节点** 

   客户端与Zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号。

***说明：创建znode时设置顺序标识，znode名称后会附加一个值，顺序号是一个单调递增的计数器，由父节点维护.***

***在分布式系统中，顺序号可以被用于为所有的事件进行全局排序， 这样客户端可以通过顺序号推断事件的顺序***



### Stat结构体

1. czxid - 创建节点的事务 zxid <br>每次修改 ZooKeeper 状态都会收到一个 zxid 形式的时间戳，也就是 ZooKeeper 事务 ID。事务 ID 是 ZooKeeper 中所有修改总的次序。每个修改都有唯一的 zxid，如果 zxid1 小于 zxid2，那么 zxid1 在 zxid2 之前发生。
2. ctime - znode 被创建的毫秒数(从 1970 年开始)
3. mzxid - znode 最后更新的事务 zxid
4. mtime - znode 最后修改的毫秒数(从 1970 年开始)
5. pZxid-znode 最后更新的子节点 zxid
6. cversion - znode 子节点变化号， znode 子节点修改次数
7. dataversion - znode 数据变化号
8. aclVersion - znode 访问控制列表的变化号
9. ephemeralOwner- 如果是临时节点，这个是 znode 拥有者的 session id。如果不是临时节点则是 0。
10. dataLength- znode 的数据长度
11. numChildren - znode 子节点数量

### 监听器原理

**面试重点**

监听原理详解：

1. 首先要有一个main()线程
2. 在main线程中创建Zookeeper客户端， 这时就会创建两个线程， 一个负责网络连接通信（ connet ）， 一个负责监听（ listener ）。
3. 通过connect线程将注册的监听事件发送给Zookeeper。
4. 在Zookeeper的注册监听器列表中将注册的监听事件添加到列表中。
5. Zookeeper监听到有数据或路径变化， 就会将这个消息发送给listener线程。
6. listener线程内部调用了process()方法。

![](/assets/imgs/10.png)

常见的监听:

1. 监听节点数据的变化 `get path [watch]`
2. 监听子节点增减的变化 `ls path [watch]`

### 写数据流程

![](/assets/imgs/11.png)

1. Client 向 ZooKeeper 的Server1 上写数据，发送一个写请求。
2. 如果Server1不是Leader，那么Server1 会把接受到的请求进一步转发给Leader，因为每个ZooKeeper的Server里面有一个是Leader。这个Leader 会将写请求广播给各个Server， 比如Server1和Server2，各个**Server会将该写请求加入待写队列**，并向Leader发送成功信息。
3. 当Leader收到半数以上 Server 的成功信息， 说明该写操作可以执行。Leader会向各个Server 发送提交信息，各个Server收到信息后会**落实队列里的写请求， 此时写成功**。
4. Server1会进一步通知 Client 数据写成功了，这时就认为整个写操作成功

## Zookerper实战 🤔

### 分布式安装

在单机上实现伪集群。

**1.docker创建容器 myzk、myzk2、myzk3**

```shell
docker run -v /root/zookeeper:/opt/module  --privileged=true --name myzk -d redis:5.0.7
docker run -v /root/zookeeper:/opt/module  --privileged=true --name myzk2 -d redis:5.0.7
docker run -v /root/zookeeper:/opt/module  --privileged=true --name myzk3 -d redis:5.0.7
docker exec -it myzk env LANG=C.UTF-8 /bin/bash
tar -zxvf zookeeper-3.4.14.tar.gz
cp -rf /opt/module/zookeeper-3.4.14 /root/silince/
cp -rf /opt/module/jdk1.8.0_251 /root/silince/
# 配置jdk环境变量 /etc/profile
export JAVA_HOME=/root/jdk/jdk1.8.0_251
export LASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
export PATH=$PATH:${JAVA_HOME}/bin
# 生效配置文件
source /etc/profile
```

**2.配置服务器编号**

```shell
cd /root/silince/zookeeper-3.4.14
# 在zookeeper目录下创建 zkData
mkdir zkData && cd zkData
# 在zkData中创建一个 myid 的文件，添加与 server 对应的编号 2
vim myid
# 并分别在 myzk2、myzk3 上修改 myid 文件中内容为 3、4
```

**3.配置 zoo.cfg 文件**

```shell
# 重命名/zookeeper-3.4.14/conf 这个目录下的 zoo_sample.cfg 为 zoo.cfg
mv zoo_sample.cfg zoo.cfg
# 修改zoo.cfg 文件数据存储路径配置 ⚠️ 虚拟机硬盘设的太小了导致没法改，吐啦
dataDir=/root/silince/zookeeper-3.4.14/zkData
# 增加如下配置
server.2=172.17.0.2:2888:3888 
server.3=172.17.0.3:2888:3888 
server.4=172.17.0.4:2888:3888
# ps 查看容器内网的ip地址等信息 
docker inspect  容器/镜像 |grep IPAddress
```

配置参数解读 `server.A=B:C:D。`

- A 是一个数字，表示这个是第几号服务器;
  - 集群模式下配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面有一个数据 就是 A 的值，Zookeeper 启动时读取此文件，拿到里面的数据与 zoo.cfg 里面的配置信息比 较从而判断到底是哪个 server。

- B 是这个服务器的地址;
- C 是这个服务器 Follower 与集群中的 Leader 服务器交换信息的端口;
- D 是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

**4.集群操作**

分别启动zookeeper

```
bin/zkServer.sh start
```

查看状态

```
bin/zkServer.sh status
```





### 客户端命令行操作

| 命令基本语法     | 功能描述                                               |
| ---------------- | ------------------------------------------------------ |
| help             | 显示所有操作命令                                       |
| ls path [watch]  | 使用 ls 命令来查看当前 znode 中所包含的内容            |
| ls2 path [watch] | 查看当前节点数据并能看到更新次数等数据                 |
| create           | 普通创建<br>-s 含有序列<br>-e 临时（重启或者超时消失） |
| get path [watch] | 获得节点的值                                           |
| set              | 设置节点的具体值                                       |
| stat             | 查看节点状态                                           |
| delete           | 删除节点                                               |
| rmr              | 递归删除节点                                           |

1． 启动客户端

	zkCli.cmd -server 127.0.0.1:2182

---

2．显示所有操作命令

	help

---

3． 查看当前 znode 中所包含的内容

	ls /
	[zookeeper]

---

4． 查看当前节点详细数据

	ls2 /
	[zookeeper]
	cZxid = 0x0
	ctime = Thu Jan 01 08:00:00 CST 1970
	mZxid = 0x0
	mtime = Thu Jan 01 08:00:00 CST 1970
	pZxid = 0x0
	cversion = -1
	dataVersion = 0
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 0
	numChildren = 1

---

5． 分别创建 2 个普通节点

	create /sanguo "jinlian"
	Created /sanguo
	
	create /sanguo/shuguo "liubei"
	Created /sanguo/shuguo

---

6．获得节点的值

	get /sanguo
	jinlian
	cZxid = 0x300000004
	ctime = Sat Jul 18 13:07:51 CST 2020
	mZxid = 0x300000004
	mtime = Sat Jul 18 13:07:51 CST 2020
	pZxid = 0x300000005
	cversion = 1
	dataVersion = 0
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 7
	numChildren = 1
	
	get /sanguo/shuguo
	liubei
	cZxid = 0x300000005
	ctime = Sat Jul 18 13:09:21 CST 2020
	mZxid = 0x300000005
	mtime = Sat Jul 18 13:09:21 CST 2020
	pZxid = 0x300000005
	cversion = 0
	dataVersion = 0
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 6
	numChildren = 0
	[zk: 127.0.0.1:2182(CONNECTED) 8]

---

7． 创建短暂节点

	create -e /sanguo/wuguo "zhouyu"
	Created /sanguo/wuguo

a. 在当前客户端是能查看到的

	ls /sanguo
	[wuguo, shuguo]

b. 退出当前客户端然后再重启客户端

	quit
	
	zkCli.cmd -server 127.0.0.1:2182

c. 再次查看根目录下短暂节点已经删除

	ls /sanguo
	[shuguo]

---

8． 创建带序号的节点

a. 先创建一个普通的根节点`/sanguo/weiguo`

	create /sanguo/weiguo "caocao"
	Created /sanguo/weiguo

b. 创建带序号的节点

	[zk: 127.0.0.1:2182(CONNECTED) 2] create -s /sanguo/weiguo/xiaoqiao "meinv"
	Created /sanguo/weiguo/xiaoqiao0000000000
	[zk: 127.0.0.1:2182(CONNECTED) 3] create -s /sanguo/weiguo/daqiao "meinv"
	Created /sanguo/weiguo/daqiao0000000001
	[zk: 127.0.0.1:2182(CONNECTED) 4] create -s /sanguo/weiguo/sunshangxiang "meinv"
	Created /sanguo/weiguo/sunshangxiang0000000002

如果原来没有序号节点，序号从 0 开始依次递增。 如果原节点下已有 2 个节点，则再排序时从 2 开始，以此类推。

---

9． 修改节点数据值

	set /sanguo/weiguo "simayi"

---

10． 节点的值变化监听


a. 在 127.0.0.1:2182 上注册监听/sanguo 节点数据变化

	[zk: 127.0.0.1:2182(CONNECTED) 7] get /sanguo watch
	jinlian
	cZxid = 0x300000004
	ctime = Sat Jul 18 13:07:51 CST 2020
	mZxid = 0x300000004
	mtime = Sat Jul 18 13:07:51 CST 2020
	pZxid = 0x300000009
	cversion = 4
	dataVersion = 0
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 7
	numChildren = 2

b. 在 127.0.0.1:2183 上修改/sanguo 节点的数据

	[zk: 127.0.0.1:2183(CONNECTED) 1] set /sanguo "tongyi"
	cZxid = 0x300000004
	ctime = Sat Jul 18 13:07:51 CST 2020
	mZxid = 0x30000000f
	mtime = Sat Jul 18 13:46:52 CST 2020
	pZxid = 0x300000009
	cversion = 4
	dataVersion = 1
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 6
	numChildren = 2
	[zk: 127.0.0.1:2183(CONNECTED) 2]

c. 观察 127.0.0.1:2182 收到数据变化的监听

	WATCHER::
	
	WatchedEvent state:SyncConnected type:NodeDataChanged path:/sanguo

---

11． 节点的子节点变化监听（路径变化）

a. 在 127.0.0.1:2182 上注册监听/sanguo 节点的子节点变化

	ls /sanguo watch
	[shuguo, weiguo]

b. 在 127.0.0.1:2183 上修改/sanguo 节点上创建子节点

	create /sanguo/jin "simayi"
	Created /sanguo/jin

c. 观察 127.0.0.1:2182 收到子节点变化的监听

	WATCHER::
	
	WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/sanguo

---

12． 删除节点

	delete /sanguo/jin
	
	get /sanguo/jin
	Node does not exist: /sanguo/jin

---

13．递归删除节点

	rmr /sanguo/shuguo
	
	get /sanguo/shuguo
	Node does not exist: /sanguo/shuguo

---

14．查看节点状态

	stat /sanguo
	cZxid = 0x300000004
	ctime = Sat Jul 18 13:07:51 CST 2020
	mZxid = 0x30000000f
	mtime = Sat Jul 18 13:46:52 CST 2020
	pZxid = 0x300000012
	cversion = 7
	dataVersion = 1
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 6
	numChildren = 1



### API应用

#### 在 Eclipse 环境搭建 ####

- 创建一个 Maven 工程
- 为 [pom.xml](pom.xml) 添加关键依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.apache.logging.log4j</groupId>
		<artifactId>log4j-core</artifactId>
		<version>2.8.2</version>
	</dependency>
	
	<dependency>
		<groupId>org.apache.zookeeper</groupId>
		<artifactId>zookeeper</artifactId>
		<version>3.4.10</version>
	</dependency>
</dependencies>
```

- 需要在项目的 `src/main/resources` 目录下，新建一个文件，命名为[log4j.properties](src/main/resources/log4j.properties)，在文件中填入如下内容：

```properties
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

#### 创建 ZooKeeper 客户端 ####

[ZooKeeperTest源码](src/test/java/com/lun/ZooKeeperTest.java)

```java
public class ZooKeeperTest {
	private static String connectString = "127.0.0.1:2182,127.0.0.1:2183,127.0.0.1:2184";
	private static int sessionTimeout = 2000;
	private ZooKeeper zkClient = null;

	// @Test
	@Before
	public void init() throws Exception {

		zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
			@Override
			// 收到事件通知后的回调函数（用户的业务逻辑）
			public void process(WatchedEvent event) {
				System.out.println(event.getType() + "--" + event.getPath());

				// 再次启动监听
				try {
					List<String> children = zkClient.getChildren("/", true);
					for (String child : children) {
						System.out.println(child);
					}
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}
}
```

#### 创建一个节点

```java
// 创建子节点
@Test
public void create() throws Exception {
	// 参数 1：要创建的节点的路径； 参数 2：节点数据 ； 参数 3：节点权限 ；参数 4：节点的类型
	String nodeCreated = zkClient.create("/root", "root".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

	System.out.println(nodeCreated);
}
```

#### 获取子节点并监听节点变化

```java
// 获取子节点
@Test
public void getChildren() throws Exception {
	List<String> children = zkClient.getChildren("/", true);
	for (String child : children) {
		System.out.println(child);
	}

	// 延时阻塞
	Thread.sleep(Long.MAX_VALUE);
}
```

#### 判断节点是否存在

```java
// 判断 znode 是否存在
@Test
public void exist() throws Exception {
	Stat stat = zkClient.exists("/eclipse", false);
	System.out.println(stat == null ? "not exist" : "exist");
}
```

### 服务器节点动态上下线案例分析

#### 需求 ####

某分布式系统中，主节点可以有多台，可以动态上下线，任意一台客户端都能实时感知到主节点服务器的上下线。

#### 需求分析 ####

![](/assets/imgs/01.png)

#### 服务器节点动态上下线案例注册代码

##### 先在集群上创建/servers节点 #####

	[zk: 127.0.0.1:2182(CONNECTED) 6] create /servers "servers"
	Created /servers

##### 服务器端向 Zookeeper 注册代码 #####

[DistributeServer源码](src/main/java/com/lun/DistributeServer.java)

```java
public class DistributeServer {
	private static String connectString = "127.0.0.1:2182,127.0.0.1:2183,127.0.0.1:2184";
	private static int sessionTimeout = 2000;
	private ZooKeeper zkClient = null;

	private String parentNode = "/servers";

	// 创建到 zk 的客户端连接
	public void getConnect() throws IOException {
		zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
			@Override
			public void process(WatchedEvent event) {
			}
		});
	}

	// 注册服务器
	public void registServer(String hostname) throws Exception {
		String create = zkClient.create(parentNode + "/server", hostname.getBytes(), Ids.OPEN_ACL_UNSAFE,
				CreateMode.EPHEMERAL_SEQUENTIAL);
		System.out.println(hostname + " is online " + create);
	}

	// 业务功能
	public void business(String hostname) throws Exception {
		System.out.println(hostname + " is working ...");
		Thread.sleep(Long.MAX_VALUE);
	}

	public static void main(String[] args) throws Exception {
		// 1 获取 zk 连接
		DistributeServer server = new DistributeServer();
		server.getConnect();
		// 2 利用 zk 连接注册服务器信息
		server.registServer(args[0]);
		// 3 启动业务功能
		server.business(args[0]);
	}

}
```

##### 服务器节点动态上下线案例全部代码实现

[DistributeClient源码](src/main/java/com/lun/DistributeClient.java)

```java
public class DistributeClient {
	private static String connectString = "127.0.0.1:2182,127.0.0.1:2183,127.0.0.1:2184";
	private static int sessionTimeout = 2000;
	private ZooKeeper zkClient = null;

	private String parentNode = "/servers";

	// 创建到 zkClient 的客户端连接
	public void getConnect() throws IOException {
		zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
			@Override
			public void process(WatchedEvent event) {
				// 再次启动监听
				try {
					getServerList();
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}

	// 获取服务器列表信息
	public void getServerList() throws Exception {
		// 1 获取服务器子节点信息，并且对父节点进行监听
		List<String> children = zkClient.getChildren(parentNode, true);
		// 2 存储服务器信息列表
		ArrayList<String> servers = new ArrayList<>();
		// 3 遍历所有节点，获取节点中的主机名称信息
		for (String child : children) {
			byte[] data = zkClient.getData(parentNode + "/" + child, false, null);
			servers.add(new String(data));
		}
		// 4 打印服务器列表信息
		System.out.println(servers);
	}

	// 业务功能
	public void business() throws Exception {
		System.out.println("client is working ...");
		Thread.sleep(Long.MAX_VALUE);
	}

	public static void main(String[] args) throws Exception {
		// 1 获取 zk 连接
		DistributeClient client = new DistributeClient();
		client.getConnect();
		// 2 获取 servers 的子节点信息，从中获取服务器信息列表
		client.getServerList();
		// 3 业务进程启动
		client.business();
	}

}
```

# 企业面试真题

**请简述 ZooKeeper 的选举机制**

半数机制，myid最大的为Leader

---

**ZooKeeper 的监听原理是什么？**

![](/assets/imgs/10.png)

---

**ZooKeeper 的部署方式有哪几种？集群中的角色有哪些？集群最少需要几台机器？**

1. 部署方式单机模式、集群模式
2. 角色： Leader 和 Follower
3. 集群最少需要机器数： 3

---

**ZooKeeper 的常用命令**

CRUD：

1. C
	- create
2. R 
	- ls
	- ls2
	- get
	- stat
3. U
	- set
4. D
	- delete
	- rmr
5. help
