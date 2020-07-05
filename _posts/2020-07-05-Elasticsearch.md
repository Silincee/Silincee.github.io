---
layout: post
title:  "Elasticsearch"
date:   2020-07-05 20:40:06 +0800--
categories: [JavaEE]
tags: [Elasticsearch, Java, ]  
---

# 😈Elasticsearch 启动指令

```
#启动ES
cd /Users/silince/Applications/ES6/elasticsearch-5.4.0
./bin/elasticsearch -d

#启动Head插件
cd /Users/silince/Applications/ES6/elasticsearch-head
grunt server

#关闭
jps
kill ...
```



# 😈在centos中启动ES6

```dockerfile
docker run -e ES_JAVA_OPTS="-Xms256 -Xmx256m" -d -p 9200:9200  -p 9300:9300 --name ES01 镜像id

-e ES_JAVA_OPTS="-Xms256 -Xmx256m"  限制内存

-d 后台运行

-p 9200:9200  -p 9300:9300 暴露端口
```





# 😈Elasticsearch核心概念

> /Users/silince/Develop/MagicDontTouch/IdeaProjects/ElasticSearch/es-first

1.集群

​		一个或者多个安装Elasticsearch的服务器节点组织在一起就是集群，他们共同持有你整个的数据，并一起提供索引和搜索功能。一个集群由一个唯一的名字标识，成为cluster name，集群名称默认是“elasticserch”。集群名字非常重要，就像一个组织的名称，具有相同集群名称的节点才会组成一个集群。集群名称可以再配置文件中指定。

2.节点

​		一个节点是你集群中的一个服务器，作为集群的一部分，它存储你的数据，参与集群的索引和搜索功能。

​		一个节点可以通过配置集群名称的方式来加加入一个指定的集群。默认情况下，每个节点都会被安排加入到一个集群名称为“elasticsearch”的集群中，这意味着如果你在你的网络汇中启动了若干个节点，并假定他们能够相互彼此发现，他们将会自动地形成并加入到一个叫elasticsearch的集群中。但是有的时候这种机制并不可靠，会发生脑裂现象，往往不如在每一个节点上配置节点的名字在启动时进行被动发现来的安全稳定。

3.索引

​		一个索引就是一个拥有几分相似特征的文档的集合，索引的数据结构仍然是倒排索引。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。一个索引由一个名字来标识（必须全部是小写的字母），并且当我们要对这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。在一个集群中，可以定义任意多的索引。

​		索引作动词的时候表示索引数据和对数据进行索引操作。

4.类型

​		在一个索引中，你可以定义一种或多种类型。一个类型是你的索引的一类逻辑上的分类或分区，其语意完全由你来定。通常，会为具有一组共同字段的文档定义一个类型。

5.文档

​		一个文档是一个可被索引的基础信息单元。比如，你可以拥有某一个客户的文档，某一个产品的文档。淡然也可以拥有某个订单的一个文档。文档都是JSON格式。

6.分片

​		一个索引可以存储超出单个节点硬件限制的大量数据。比如一个具有10亿文档的索引占据1TB的磁盘空间，而任意节点都没有这样大的磁盘空间；或者单个节点处理搜索请求，响应太慢。为了解决这个问题，ELasticsearch提供了将索引划分成多份的能力，这些份就叫做分片。当你创建一个索引的时候，可以指定想要的分片的数量，每个分片本身也是一个功能完整并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。

7.副本

​		在一个网络/云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于离线状态，或者由于某种原因消失了，这种情况下，有一个故障转移机制非常有用，并且也是强烈推荐的。为此，Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片，或者直接叫做副本。

​		总之，每个索引可以被分为多个分片。一个索引可以有一至多个副本。一旦有了副本，每个索引就有了主分片（作为复制源的原来的分片）和副本分片（主分片的拷贝）之别。分片和副本的数量可以在索引创建的时候指定。在索引创建之后，可以在任何时候动态地改变副本的数量，但是事后不能改变分片的数量。



# 😈Spring Data Elasticsearch

> /Users/silince/Develop/MagicDontTouch/IdeaProjects/ElasticSearch/springdata-elasticsearch

### 1. 什么是Spring Data Elasticsearch

Spring Data 是一个用于简化数据库访问，并支持云服务的开源框架。其主要目标是使得对数据的访问变得方便快捷，并支持map-reduce框架和云计算数据服务。Spring Data可以极大的简化JPA的写法，可以在几乎不用写实现的情况下，实现对数据的访问和操作。除了CRUD外，还包括如分页、排序等一些常用的功能。

### 2. 搭建工程

1. 创建一个pom工程
2. 添加相关坐标
3. 创建一个spring配置文件
   1. 配置 `elasticsearch:transport-client`
   2. `elasticsearch:repositories` 包扫描器，扫描dao
   3. 配置 `ElasticsearchTemplate`对象，就是一个bean

### 3. 管理索引库

1. 创建一个实体类，其实就是一个JavaBean(pojo)映射到一个文档上，需要添加一些注解进行标注
2. 创建一个Dao，需要继承ElasticSearchRepository接口。
3. 编写测试代码

### 4. 创建索引

直接使用ElasticsearchTemplate对象的createIndex方法创建索引，并配置映射关系

### 5. 添加文档

1. 创建一个Article对象
2. 使用ArticleRepository对象向索引库中添加文档 。

### 6. 删除文档

直接使用ArticleRepository对象的deleteById方法直接删除

### 7. 查询文档

直接使用ArticleRepository对象的查询方法

### 8. 自定义查询方法

根据SpringDataES的命名规则来命名，无需自己实现。

ps：

1. 如果不设置分页信息，默认带分页，每页显示10条数据。设置分页使用在方法中添加参数Pageable，从第0页开始      `Pageable pageable = PageRequest.of(0,15);`

2. 可以对搜索的内容先分词然后再进行查询，每个词之间都是and的关系

### 9. 使用原生的查询条件查询

1. 创建NativeSearchQuery对象，设置查询条件(QueryBuilder)
2. 使用ElasticSearchTemplate对象执行查询
3. 取查询结果



