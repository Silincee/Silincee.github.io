---
layout: post
title:  "ConcurrentHashMap源码分析"
date:   2020-12-19 13:16:06 +0800--
categories: [Java, 高并发]
tags: [HashMap, 高并发, 集合框架, ]  
---

# 多线程下的 HashMap 存在的问题

HashMap 在多线程下是不安全的。但是，在JDK1.8 的 HashMap 改为采用尾插法，已经不存在死循环的问题了，为什么也会线程不安全呢？我们以 put 方法为例（1.8）：

**假如现在有两个线程都执行到了下图的红色箭头处。当线程一判断为空之后，CPU 时间片到了，被挂起。线程二也执行到此处判断为空，继续执行下一句，创建了一个新节点，插入到此下标位置。然后，线程一解挂，同样认为此下标的元素为空，因此也创建了一个新节点放在此下标处，因此造成了元素的覆盖。**

所以，可以看到不管是 JDK1.7 还是 1.8 的 HashMap 都存在线程安全的问题。那么，在多线程环境下，应该怎样去保证线程安全呢？

![image-20210119153611212](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210119153611212.png)





# 为什么选用 ConcurrentHashMap

首先，你可能想到，在多线程环境下用 Hashtable 来解决线程安全的问题。这样确实是可以的，但是同样的它也有缺点，我们看下最常用的 put 方法和 get 方法。

- Hashtable-put()

![image-20210119154636994](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210119154636994.png)

- Hashtable-get()

![image-20210119154708457](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210119154708457.png)

可以看到，不管是往 map 里边添加元素还是获取元素，都会用 `synchronized` 关键字加锁。**当有多个元素之前存在资源竞争时，只能有一个线程可以获取到锁，操作资源。更不能忍的是，一个简单的读取操作，互相之间又不影响，为什么也不能同时进行呢？**

所以，hashtable 的缺点显而易见，它不管是 get 还是 put 操作，都是锁住了整个 table，效率低下，因此 **并不适合高并发场景**。

**也许，你还会想起来一个集合工具类 Collections，生成一个SynchronizedMap。其实，它和 Hashtable 差不多，同样的原因，锁住整张表，效率低下。**

![Image](/assets/imgs/640-0982385..png)

所以，思考一下，既然锁住整张表的话，并发效率低下，那我把整张表分成 N 个部分，并使元素尽量均匀的分布到每个部分中，分别给他们加锁，互相之间并不影响，这种方式岂不是更好 :

- 这就是在 JDK1.7 中 ConcurrentHashMap 采用的方案，被叫做**锁分段技术**，每个部分就是一个 Segment（段）。

- 但是，在JDK1.8中，完全重构了，**采用的是 `Synchronized + CAS` ，把锁的粒度进一步降低**，而放弃了 Segment 分段。（此时的 Synchronized 已经升级了，效率得到了很大提升，锁升级可以了解一下）



# ConcurrentHashMap 1.7 源码解析

我们看下在 JDK1.7中 ConcurrentHashMap 是怎么实现的。在本文之前了解一下多线程的基本知识，如[JMM内存模型](http://www.silince.cn/2020/12/12/谈谈Volatile/#jmm概述)，[volatile关键字作用](http://www.silince.cn/2020/12/12/谈谈Volatile/#可见型)，[CAS和自旋](www.silince.cn/2020/09/01/谈谈CAS/)，[ReentranLock重入锁](http://www.silince.cn/2020/09/03/从ReentrantLock的实现看AQS的原理/)。

## 底层存储结构

在 JDK1.7中，本质上还是采用`链表+数组`的形式存储键值对的。但是，为了提高并发，把原来的整个 table 划分为 n 个 Segment 。所以，从整体来看，它是一个由 Segment 组成的数组。然后，每个 Segment 里边是由 HashEntry 组成的数组，每个 HashEntry之间又可以形成链表。**我们可以把每个 Segment 看成是一个小的 HashMap，其内部结构和 HashMap 是一模一样的。**

![image-20210119155736971](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210119155736971.png)

当对某个 Segment 加锁时，如图中 Segment2，并不会影响到其他 Segment 的读写。每个 Segment 内部自己操作自己的数据。这样一来，我们要做的就是尽可能的让元素均匀的分布在不同的 Segment中。最理想的状态是，所有执行的线程操作的元素都是不同的 Segment，这样就可以降低锁的竞争。

废话了这么多，还是来看底层源码吧，因为所有的思想都在代码里体现。



## 常用变量

先看下 1.7 中常用的变量和内部类都有哪些，这有助于我们了解 ConcurrentHashMap 的整体结构

```java
//默认初始化容量，这个和 HashMap中的容量是一个概念，表示的是整个 Map的容量
static final int DEFAULT_INITIAL_CAPACITY = 16;

//默认加载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//默认的并发级别，这个参数决定了 Segment 数组的长度
static final int DEFAULT_CONCURRENCY_LEVEL = 16;

//最大的容量
static final int MAXIMUM_CAPACITY = 1 << 30;

//每个Segment中table数组的最小长度为2，且必须是2的n次幂。
//由于每个Segment是懒加载的，用的时候才会初始化，因此为了避免使用时立即调整大小，设定了最小容量2
static final int MIN_SEGMENT_TABLE_CAPACITY = 2;

//用于限制Segment数量的最大值，必须是2的n次幂
static final int MAX_SEGMENTS = 1 << 16; // slightly conservative

//在size方法和containsValue方法，会优先采用乐观的方式不加锁，直到重试次数达到2，才会对所有Segment加锁
//这个值的设定，是为了避免无限次的重试。后边size方法会详讲怎么实现乐观机制的。
static final int RETRIES_BEFORE_LOCK = 2;

//segment掩码值，用于根据元素的hash值定位所在的 Segment 下标。后边会细讲
final int segmentMask;

//和 segmentMask 配合使用来定位 Segment 的数组下标，后边讲。
final int segmentShift;

// Segment 组成的数组，每一个 Segment 都可以看做是一个特殊的 HashMap
final Segment<K,V>[] segments;

//Segment 对象，继承自 ReentrantLock 可重入锁。
//其内部的属性和方法和 HashMap 神似，只是多了一些拓展功能。
static final class Segment<K,V> extends ReentrantLock implements Serializable {

    //这是在 scanAndLockForPut 方法中用到的一个参数，用于计算最大重试次数
    //获取当前可用的处理器的数量，若大于1，则返回64，否则返回1。
    static final int MAX_SCAN_RETRIES =
      Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;

    //用于表示每个Segment中的 table，是一个用HashEntry组成的数组。
    transient volatile HashEntry<K,V>[] table;

    //Segment中的元素个数，每个Segment单独计数（下边的几个参数同样的都是单独计数）
    transient int count;

    //每次 table 结构修改时，如put，remove等，此变量都会自增
    transient int modCount;

    //当前Segment扩容的阈值，同HashMap计算方法一样也是容量乘以加载因子
    //需要知道的是，每个Segment都是单独处理扩容的，互相之间不会产生影响
    transient int threshold;

    //加载因子
    final float loadFactor;

    //Segment构造函数
    Segment(float lf, int threshold, HashEntry<K,V>[] tab) {
      this.loadFactor = lf;
      this.threshold = threshold;
      this.table = tab;
    }
    ...
      // put(),remove(),rehash() 方法都在此类定义
}

// HashEntry，存在于每个Segment中，它就类似于HashMap中的Node，用于存储键值对的具体数据和维护单向链表的关系
static final class HashEntry<K,V> {
    //每个key通过哈希运算后的结果，用的是 Wang/Jenkins hash 的变种算法，此处不细讲，感兴趣的可自行查阅相关资料
    final int hash;
    final K key;
    //value和next都用 volatile 修饰，用于保证内存可见性和禁止指令重排序
    volatile V value;
    //指向下一个节点
    volatile HashEntry<K,V> next;

    HashEntry(int hash, K key, V value, HashEntry<K,V> next) {
      this.hash = hash;
      this.key = key;
      this.value = value;
      this.next = next;
  }
}
```



## 构造函数

ConcurrentHashMap 有五种构造函数，但是最终都会调用同一个构造函数，所以只需要搞明白这一个核心的构造函数就可以了。

```java
public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
   //检验参数是否合法。值得说的是，并发级别一定要大于0，否则就没办法实现分段锁了。
   if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
    	throw new IllegalArgumentException();
   //并发级别不能超过最大值
   if (concurrencyLevel > MAX_SEGMENTS)
    	concurrencyLevel = MAX_SEGMENTS;
   // Find power-of-two sizes best matching arguments
   //偏移量，是为了对hash值做位移操作，计算元素所在的Segment下标，put方法详讲
   int sshift = 0;
   //用于设定最终Segment数组的长度，必须是2的n次幂
   int ssize = 1;
   //这里就是计算 sshift 和 ssize 值的过程  1⃣️
   while (ssize < concurrencyLevel) {
      ++sshift;
      ssize <<= 1;
   }
   this.segmentShift = 32 - sshift;
   //Segment的掩码
   this.segmentMask = ssize - 1;
   if (initialCapacity > MAXIMUM_CAPACITY)
    initialCapacity = MAXIMUM_CAPACITY;
   //c用于辅助计算cap的值   2⃣️
   int c = initialCapacity / ssize;
   if (c * ssize < initialCapacity)
    	++c;
   // cap 用于确定某个Segment的容量，即Segment中HashEntry数组的长度
   int cap = MIN_SEGMENT_TABLE_CAPACITY;
   // 
   while (cap < c)
    cap <<= 1;
   // create segments and segments[0]
   //这里用 loadFactor做为加载因子，cap乘以加载因子作为扩容阈值，创建长度为cap的HashEntry数组，
   //三个参数，创建一个Segment对象，保存到S0对象中。后边在 ensureSegment 方法会用到S0作为原型对象去创建对应的Segment。
   Segment<K,V> s0 =
    new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
         (HashEntry<K,V>[])new HashEntry[cap]);
   //创建出长度为 ssize 的一个 Segment数组
   Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
   //把S0存到Segment数组中去。在这里，我们就可以发现，此时只是创建了一个Segment数组，
   //但是并没有把数组中的每个Segment对象创建出来，仅仅创建了一个Segment用来作为原型对象。
   UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
   this.segments = ss;
}    
 //须是2的n次幂
 int ssize = 1;

上边的注释中留了 (1)(2)(3) 三个地方还没有细说。我们现在假设一组数据，把涉及到的几个变量计算出来，就能明白这些参数的含义了。

//java
//假设调用了默认构造，都用的是默认参数，即 initialCapacity 和 concurrencyLevel 都是16
//1⃣️  sshift 和 ssize 值的计算过程为，每次循环，都会把 sshift 自增1，并且 ssize 左移一位，即乘以2，
//直到 ssize 的值大于等于 concurrencyLevel 的值 16。
sshfit=0,1,2,3,4
ssize=1,2,4,8,16
//可以看到，初始他们的值分别是0和1，最终结果是4和16
//sshfit是为了辅助计算segmentShift值，ssize是为了确定Segment数组长度。
//2⃣️  此时,计算c的值，
c = 16/16 = 1;
//判断 c * 16 < 16 是否为真，真的话 c 自增1，此处为false，因此 c的值为1不变。
//3⃣️  此时，由于c为1， cap为2 ，因此判断 cap < c 为false，最终cap为2。
//总结一下，以上三个步骤，最终都是为了确定以下几个关键参数的值，
//确定 segmentShift ，这个用于后边计算hash值的偏移量，此处即为 32-4=28，
//确定 ssize，必须是一个大于等于 concurrencyLevel 的一个2的n次幂值
//确定 cap，必须是一个大于等于2的一个2的n次幂值
//感兴趣的小伙伴，还可以用另外几组参数来计算上边的参数值，可以加深理解参数的含义。
//例如initialCapacity和concurrencyLevel分别传入10和5，或者传入33和16
```



## put() 方法

put 方法的总体流程是：

1. 通过哈希算法计算出当前 key 的 hash 值
2. 通过这个 hash 值找到它所对应的 Segment 数组的下标
3. 再通过 hash 值计算出它在对应 Segment 的 HashEntry数组 的下标
4. 找到合适的位置插入元素

```java
/********************这是Map的put方法********************/
public V put(K key, V value) {
   Segment<K,V> s;
   //不支持value为空
   if (value == null)
    throw new NullPointerException();
   //通过 Wang/Jenkins 算法的一个变种算法，计算出当前key对应的hash值
   int hash = hash(key);
   //上边我们计算出的 segmentShift为28，因此hash值右移28位，说明此时用的是hash的高4位，
   //然后把它和掩码15进行与运算，得到的值一定是一个 0000 ~ 1111 范围内的值，即 0~15 。
   int j = (hash >>> segmentShift) & segmentMask;
   //这里是用Unsafe类的原子操作找到Segment数组中j下标的 Segment 对象
   if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
     (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
    //初始化j下标的Segment
    s = ensureSegment(j);
   //在此Segment中添加元素
   return s.put(key, hash, value, false);
}
```

上边有一个这样的方法， `UNSAFE.getObject (segments, (j << SSHIFT) + SBASE`。它是为了通过Unsafe这个类，找到 j 最新的实际值。这个计算` (j << SSHIFT) + SBASE` ，在后边非常常见，我们只需要知道它代表的是 j 的一个偏移量，通过偏移量，就可以得到 j 的实际值。可以类比，AQS 中的 CAS 操作。Unsafe中的操作，都需要一个偏移量，看下图，

![image-20210119184245176](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210119184245176.png)

`(j << SSHIFT) + SBASE` 就相当于图中的 stateOffset偏移量。只不过图中是 CAS 设置新值，而我们这里是取 j 的最新值。后边很多这样的计算方式，就不赘述了。接着看 s.put 方法，这才是最终确定元素位置的方法。

```java
/********************这是Map的put方法********************/
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
  //这里通过tryLock尝试加锁，如果加锁成功，返回null，否则执行 scanAndLockForPut方法
  //这里说明一下，tryLock 和 lock 是 ReentrantLock 中的方法，
  //区别是 tryLock 不会阻塞，抢锁成功就返回true，失败就立马返回false，
  //而 lock 方法是，抢锁成功则返回，失败则会进入同步队列，阻塞等待获取锁。
  HashEntry<K, V> node = tryLock() ? null :
  scanAndLockForPut(key, hash, value);
  V oldValue;
  try {
    //当前Segment的table数组
    HashEntry<K, V>[] tab = table;
    //这里就是通过hash值，与tab数组长度取模，找到其所在HashEntry数组的下标
    int index = (tab.length - 1) & hash;
    //当前下标位置的第一个HashEntry节点
    HashEntry<K, V> first = entryAt(tab, index);
    for (HashEntry<K, V> e = first; ; ) {
      //如果第一个节点不为空
      if (e != null) {
        K k;
        //并且第一个节点，就是要插入的节点，则替换value值，否则继续向后查找
        if ((k = e.key) == key ||
            (e.hash == hash && key.equals(k))) {
          //替换旧值
          oldValue = e.value;
          if (!onlyIfAbsent) {
            e.value = value;
            ++modCount;
          }
          break;
        }
        e = e.next;
      }
      //说明当前index位置不存在任何节点，此时first为null，
      //或者当前index存在一条链表，并且已经遍历完了还没找到相等的key，此时first就是链表第一个元素
      else {
        //如果node不为空，则直接头插
        if (node != null)
          node.setNext(first);
        //否则，创建一个新的node，并头插
        else
          node = new HashEntry<K, V>(hash, key, value, first);
        int c = count + 1;
        //如果当前Segment中的元素大于阈值，并且tab长度没有超过容量最大值，则扩容
        if (c > threshold && tab.length < MAXIMUM_CAPACITY)
          rehash(node);
        //否则，就把当前node设置为index下标位置新的头结点
        else
          setEntryAt(tab, index, node);
        ++modCount;
        //更新count值
        count = c;
        //这种情况说明旧值肯定为空
        oldValue = null;
        break;
      }
    }
  } finally {
    //需要注意ReentrantLock必须手动解锁
    unlock();
  }
  //返回旧值
  return oldValue;
}
```





## ensureSegment() 方法

## scanAndLockForPut() 方法

## rehash() 扩容机制

## get() 获取元素方法

## remove() 方法

## size() 方法是怎么统计元素个数的





# ConcurrentHashMap 1.8 源码解析

## put()方法详解

## initTable()初始化表

## addCount()方法

## fullAddCount()方法

## transfer()是怎样扩容和迁移元素的

## helpTransfer()方法帮助迁移元素