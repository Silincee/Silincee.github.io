---
layout: post
title:  "Elasticsearch中的数值类型索引"
date:   2022-03-03 21:10:06 +0800--
categories: [Java, ]
tags: [Elasticsearch, ]  
---

# 前言

最近杂七杂八的事情比较多，好久没更新文章了🤦‍♀️，今天就好好来理一理之前没搞清楚的关于ES数值索引的问题。ES主要是用于解决文本检索的场景，ES会默认将所有的输入内容当作字符串来理解，对于字段类型是keyword或者text的数据比较友好。但是如果输入的类型是数字，ES还会把数字当作字符串吗？排序问题还有范围查询问题怎么解决呢？



# 为什么要用BKD Tree

从Elasticsearch5.x开始，Elasticsearch开始使用Lucene6.0版本，而Lucene6.0版本对于Lucene来说有非常大的改变，随之带来的是Elasticsearch有很大的改变。

ES使用的搜索库Lucene在6.0版本以及以后为了解决**多维空间位置**搜索问题，改用新的数据结构——**BKD树**来实现位置搜索，带来了很大的性能提升。开发者发现这种实现也能用于**一维**数据搜索，于是用新的数据结构代替了现在的**字符串的实现**。

## kd树

k-d树(k-dimensional)，也就是k维树.

**kd树是每个叶子节点都为k维点的二叉树。所有非叶子节点可以视作用一个超平面把空间分割成两个半空间。节点左边的子树代表在超平面左边的点，节点右边的子树代表在超平面右边的点。**选择超平面的方法如下：每个节点都与k维中垂直于超平面的那一维有关。因此，如果选择按照x轴划分，所有x值小于指定值的节点都会出现在左子树，所有x值大于指定值的节点都会出现在右子树。这样，超平面可以用该x值来确定，其法线为x轴的单位向量。

kd树允许节点内部有任意的维度的数据。kd树使用二叉树实现，类似于一个BST。关键的不同是，当每个节点比较大小时，可以使用不同的维度来比较。下图是一个2k树的结构与其对应的二维空间，它使用了不同的维度来划分左右区域。

![image-20220303220207287](/assets/imgs/image-20220303220207287.png)

优缺点：

- 优点：K-D树和BST一样不仅可以精确查找，也更适合做范围查询，但K-D树比BST更强，**它能对多个维度进行范围查询。**

- 缺点：类似于二叉搜索树，如果一个kd树是平衡的，可以保证O(logn) 的时间复杂度，因此每一个节点都把整个集合划分成了两半。关键的问题就是，只有平衡的情况下才能保证这一点。设想一下，给图中的kd树，添加两个节点(1,1) ,(0,0), 整棵树的所有节点几乎已经全部在左侧了，这样就破坏了原有的平衡。更复杂的是，对于kd树而言，没有一些标准的技术来让树恢复平衡，比如树的旋转等操作。我们能做的就是完全的重建这棵树。**因此，标准的kd树对于动态的更新，不提供很好的性能，只有在静态数据集上，kd树才有很好的性能.**



## kdb树

接下来的进阶版本是`KDB树`(k-dimensional B-Tree)，这种数据结构混合了kd树和B+树之后会拿到的结构。和标准的kd树一样，一个内部节点将空间分为两半。不过和kd树不一样的是，kdb树的内部节点不存储它们自己的数据。空间内会两个点定义个区域，每个维度的第一个点定义了最小值，第二个点定义了最大值。

因此K-D-B树具备了B树的优点：**提供平衡K-D树的搜索效率，同时提供和B树一样面向块的存储以优化外存设备的访问**。

<img src="/assets/imgs/image-20220303220631303.png" alt="image-20220303220631303" style="zoom:50%;" />

由于kdb树存储表现是一颗B树，他在磁盘上的性能很好。这是因为**提高了每个节点的扇出率(内部节点不存储数据)**，导致节点变大以及树变矮。磁盘通常有高的延迟及高的吞吐量，这意味着读取大的数据块是有优势的，因为读取小的数据块将话费更多的时间在延迟上。树比较低，意味着需要更少的逻辑读取。在磁盘上，一个b树的节点的大小至少是和一个页一样大，也就是4k，更多的时间是大于这个值的。因此，一个节点经常有成百上千个孩子节点。

**像其他b树的变种一样，kdb树保证自身是平衡的树。这是通过插入策略来保证的**，如果一个插入元素在叶子节点上，且这个叶子节点没满, 那么将这个元素添加到这个叶子节点。如果叶子节点是满的，那就进行分裂。不是通常想象的将值写入到树的高层节点上，而是只给高层节点添加一个区域。如果一个元素在区域之外，事情就更复杂了。

与B树这类存储一维数据（数据只按一种规则排序）的数据结构不同，K-D-B树数据有多个维度，所以页分裂的时候不能以B树“一刀切”的方式解决。以二维K-D-B树为例，一个节点数据过多，需要分裂时，就需要拆分区域。拆分区域的过程中大概率会出现拆分该区域的子区域。**这是kdb树的主要缺点**：

- 切割一个区域，通常需要切割他的孩子节点。 这个改动修改了树的大部分，如果此时我们需要写入磁盘，那么就会变得很慢。

- kdb树的另外一个缺点是**空间利用率**，由于没有约束节点的大小，可能有很大的一部分空间都浪费了。这不仅影响磁盘页的大小，还会导致更少的页被缓存到内存中。
- 所以和K-D树一样，**K-D-B树对于修改频繁的动态数据并不友好**。



## BKD树

bkd树用来解决空间问题和插入的效率问题。bkd树由多个修改后的kd树和独特的插入方法构成的，下图是一个bkd树的基础结构。

<img src="/assets/imgs/image-20220303224111845.png" alt="image-20220303224111845" style="zoom:50%;" />

这个树由一个二叉树和一个b+树构成，特殊的是，内部节点必须是一个完全二叉树。

**因为这是一颗完全二叉树，因此节点不需要记录子节点的指针，而是可以通过完全二叉树的性质算出来(假如一个节点的位置在 i ，那这个节点的左节点在位置2 i ，右节点在2 i + 1)。**更小的节点意味着可以有更多的节点被缓存在内存。同时，可以让大量的节点是叶子节点，这有效的降低了树的整体高度.

一个更大的因素，也是bkd最令人感兴趣的地方时，**这些内部节点不会被更改，bkd树使用了一种策略来添加新数据**：刚开始时，内存中缓存M个元素，这个缓冲区通常是一个简单的数组，因此他有很好的查询性能，论文中没有特别说明这个缓冲区的大小，但是按照常识来猜测，应该也是大于kd树的数量的。

<img src="/assets/imgs/image-20220303224756783.png" alt="image-20220303224756783" style="zoom:50%;" />

给定一个有N个节点的bkd树，就有log2(n/m)个改良的kd树，每一个树的节点是上一个树的两倍。**对于bkd的插入，新节点首先被插入到内存里的缓冲区里，一旦缓冲区满了，先定位到第1个为空的树。这个Buffer的数据，以及空树之前所有节点的数据一起生成一个完全二叉树。** 论文中讲了一个构建树的快速方法，这里就不再具体展开。



# Elasticsearch的filter查询

如果Elasticsearch中查询条件的多个filter，但实际上并像mysql那样按照每个filter的顺序执行，而是每一个filter都是单独执行的，最后把每个filter的结果集进行合并。Elasticsearch有个cost函数，它会先估算每个filter的代价，然后从代价最低的开始迭代docid，并且根据term、and、or的不同，来决定不同的合并方式。**合并结果集Lucene使用了跳表。**

## 数值类型的TermQuery

先从term开始说，一般term都是keyword的类型。这种方式cost则直接返回倒排中的文档频率，频率越高的迭代代价自然就越大。所以会选择频率最低的term作为开始点。因为又引入了跳表数据结构，所以效率非常的快。**postings list中的docid是顺序的，这是使用跳表的前提，因此，即使对于多个postings list的合并在磁盘上操作也不会产生太多随机IO。**在Elasticsearch5.1.1之后，取消了TermQuery的cache。就是说所有term查询，都是直接读磁盘上的数据。因为跳表和页面缓存的存在，直接在磁盘上做合并也非常快，这么做也是为了减少资源开销，提高性能。但这也带来了一个隐患。

上面说到了对于TermQuery，取消了cache，每次都去操作磁盘。虽然是对TermQuery进行了优化，但这主要是针对keyword类型，对于数值类型，**Lucene6.0以后取消了对数值类型进行倒排索引，而是采用BKD tree的方式。所以所有数值类型的TermQuery会被转成PointRangeQuery**。

<img src="/assets/imgs/image-20220303233428122.png" alt="image-20220303233428122" style="zoom:50%;" />

ES中BKD tree结构图如上所示，图中我们可以看到，**每个block中value是有序的，而docid是无序的。无序也就不能使用跳表结构。这种情况下要对结果集进行合并，就要对docidlist做一个特殊处理：从源码中看到在创建Scorer的过程中有一个bitset的迭代器，这个过程是导致查询缓慢的罪魁祸首。因为如果range的结果集范围非常大的话，耗费的时间就会越长。**

```java
@Override
public Scorer get(long leadCost) throws IOException {
  if (values.getDocCount() == reader.maxDoc()
      && values.getDocCount() == values.size()
      && cost() > reader.maxDoc() / 2) {
    // If all docs have exactly one value and the cost is greater
    // than half the leaf size then maybe we can make things faster
    // by computing the set of documents that do NOT match the range
    final FixedBitSet result = new FixedBitSet(reader.maxDoc());
    result.set(0, reader.maxDoc());
    int[] cost = new int[] { reader.maxDoc() };
    values.intersect(getInverseIntersectVisitor(result, cost));
    final DocIdSetIterator iterator = new BitSetIterator(result, cost[0]);
    return new ConstantScoreScorer(weight, score(), iterator);
  }

```

## 数值类型的RangeQuery

上面我们了解到了对数值类型的range的优化，依然达不到我们想要的效果，于是Elasticsearch从5.4版本开始又加入了 indexOrDocValuesQuery 。

这个Query包装了上面的PointRangeQuery和SortedSetDocValuesRangeQuery，并且会根据Rang查询的数据集大小，以及要做的合并操作类型，决定用哪种Query。

- 如果Range的代价小，可以用来引领合并过程，就走PointRangeQuery，直接构造bitset来进行迭代。 
- 而如果range的代价高，构造bitset太慢，就使用SortedSetDocValuesRangeQuery。这个Query利用了DocValues这种全局docid，并包含每个docid对应value的数据结构来做文档的匹配。 当给定一个docid的时候，一次随机磁盘访问就可以定位到该id对应的value，从而可以判断该doc是否match。这样从其他过滤条件返回的小结果集中一次调用docid来判断value是否匹配。



# 总结

本文我们了解了Elasticsearch和Lucene不同版本中对数值类型range查询的优化，我们牢记以下两点来避免不必要的缓慢查询。

- 对于数值类型的数据，我们是要确定该字段是需要range还是term。比如id，有些情况下我们只需要对它做term查询，这种情况下使用keyword来存储会更好。
- 如果需要使用数值类型（非keyword），并且range的结果集很大的情况下，建议升级到Elasticsearch5.4+之后版本。



# 参考

- [Multi-dimensional points, coming in Apache Lucene 6.0](https://www.elastic.co/cn/blog/lucene-points-6-0)
- [Elasticsearch干货（三）：对于数值类型索引优化](https://liubowen.blog.csdn.net/article/details/82706190?spm=1001.2101.3001.6650.6&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6.queryctrv4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6.queryctrv4&utm_relevant_index=13)
- [Lucene系列(16)工具类之kdb Bkd树原理概述](http://huyan.couplecoders.tech/%E6%90%9C%E7%B4%A2/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/bkd%E6%A0%91/2021/04/01/lucene%E7%B3%BB%E5%88%97(16)%E5%B7%A5%E5%85%B7%E7%B1%BB%E4%B9%8Bkdb-bkd%E6%A0%91%E5%8E%9F%E7%90%86%E6%A6%82%E8%BF%B0/)
