---
layout: post
title:  "从ReentrantLock的实现看AQS的原理"
date:   2020-09-03 18:58:06 +0800--
categories: [Java, 并发编程]
tags: [并发编程, ]  
---

[美团 从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)

[JUC AQS ReentrantLock源码分析（一）](https://blog.csdn.net/java_lyvee/article/details/98966684)



# 自己实现一个锁

## 方法一：通过**自旋**实现一个锁

```java
public class SpinLock {
    //原子引用线程
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void mylock() {
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName() + "\t come in");
        // 自旋获取锁
        while (!atomicReference.compareAndSet(null, thread)) {

        }
    }

    public void myUnlock() {
        Thread thread = Thread.currentThread();
        // CAS解锁
        atomicReference.compareAndSet(thread, null);
        System.out.println(Thread.currentThread().getName()+"\t invoked myunlock()");
    }
}
```

缺点：耗费CPU资源，没有竞争到锁的线程会一直占用CPU资源进行CAS操作。

## 方法二：改进为**park+自旋**实现锁：

Java提供了一个较为底层的并发工具类：**LockSupport**，可以让线程停止下来(**阻塞**)，还可以唤醒线程。

```java
// 阻塞线程
LockSupport.park(Object blocker) 
// 唤醒线程
LockSupport.unpark(Thread thread)
```

```java
public class SpinLock {
    // 锁状态
    volatile int status=0;
    // 阻塞线程队列
    Queue<Thread> parkQueue = new LinkedBlockingQueue<>();

    public void mylock() {
        // 自旋获取锁
        while (!compareAndSet(0,1)) { // 比较交换原子操作：当前为0则与1交换；当前为1则交换失败
            park();
        }
      lock()
      // 业务代码   
      ...
      // 解锁
      unlock()
    }

    public void myUnlock() {
        // CAS解锁
        status = 0;
        lock_notify();
    }
    
    public void park() {
      	// 把当前线程加入到等待队列
        parkQueue.add(Thread.currentThread());
        // 将当前线程释放CPU 阻塞
        LockSupport.park(Thread.currentThread());
    }
    
    public void lock_notify() {
        // 得到要唤醒的线程 头部线程
      	Thread t = parkQueue.header();
      	// 唤醒等待线程
      	unpack(t);
    }
}
```



# 公平与非公平锁

ReentrantLock的空参构造默认实现非公平锁，非公平锁在加锁时先进行了一次CAS获取锁的尝试，如果获取到锁，直接执行，不需要排队阻塞。***不过也可以调用其他构造实现公平锁。***

**非公平锁：**

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

**公平锁：**

```java
// Class FairSync
final void lock() {
    acquire(1);
}

// ---> acquire(1)来自于父类 AbstractQueuedSynchronizer
public final void acquire(int arg) {
  // 在 CAS 之前加了判断
  if (!tryAcquire(arg) && //⚠️  是 非tryAcquire(arg)
      acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

刚释放锁的线程再次获取同步状态的几率会非常大，使得其他线程只能在同步队列中等待。

**公平性锁保证了锁的获取按照FIFO原则，而代价是进行大量的线程切换。非公平性锁虽然可能造成线程“饥饿”，但极少的线程切换，保证了其更大的吞吐量。**



# 队列同步器AQS

ReentrantLock并发机制和操作系统的PV机制原理相同。都需要一个整形的信号量，一个等待队列，以及原子性的P操作和V操作。只不过PV操作原子性由OS保证。而AQS的原子性由CAS去保证。

***队列同步器AbstractQueuedSynchronizer（AQS）是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态***，通过内置的**FIFO队列**来完成资源获取线程的排队工作，并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。

> ReentrantLock的lock方法调用的是Sync的lock()，而Sync继承于AbstractQueuedSynchronizer。

![image-20200904165704239](/assets/imgs/image-20200904165704239.png)

## FIFO队列

同步器依赖内部的同步队列（一个FIFO双向队列）来完成同步状态的管理，当前线程获取同步状态失败时，***同步器会将当前线程以及等待状态等信息构造成为一个节点（Node）并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。***

![image-20200904161509333](/assets/imgs/image-20200904161509333.png)

## AQS的实现

### AQS框架

![img](/assets/imgs/82077ccf14127a87b77cefd1ccf562d3253591.png)

- 上图中有颜色的为Method，无颜色的为Attribution。
- 总的来说，AQS框架共分为五层，自上而下由浅入深，从AQS对外暴露的API到底层基础数据。
- 当有自定义同步器接入时，只需重写第一层所需要的部分方法即可，不需要关注底层具体的实现流程。当自定义同步器进行加锁或者解锁操作时，先经过第一层的API进入AQS内部方法，然后经过第二层进行锁的获取，接着对于获取锁失败的流程，进入第三层和第四层的等待队列处理，而这些处理方式均依赖于第五层的基础数据提供层。



**AQS的属性：**

```java
public abstract class AbstractQueuedSynchronizer {
  // 等待队列头结点
  // 将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会被序列化
	private transient volatile Node head;
  // 等待队列尾结点
	private transient volatile Node tail;
  // 状态
	private volatile int state;
  // 当前持有锁的线程。继承于AbstractOwnableSynchronizer
  private transient Thread exclusiveOwnerThread;
}
```

**AQS中的节点Node：**

```java
static final class Node {
    // 等待状态，若值为-1，表示后继节点处于等待状态
    volatile int waitStatus;
    // 前一个节点
    volatile Node prev;
    // 下一个节点
    volatile Node next;
    // ⚠️ 节点绑定线程
    volatile Thread thread;
  	// 当前线程的状态
  	volatile int waitStatus;
}
```

![image-20200904182446859](/assets/imgs/image-20200904182446859-9215167.png)

waitStatus有下面几个枚举值：

| 枚举      | 含义                                           |
| :-------- | :--------------------------------------------- |
| 0         | 当一个Node被初始化的时候的默认值               |
| CANCELLED | 为1，表示线程获取锁的请求已经取消了            |
| CONDITION | 为-2，表示节点在等待队列中，节点线程等待唤醒   |
| PROPAGATE | 为-3，当前线程处在SHARED情况下，该字段才会使用 |
| SIGNAL    | 为-1，表示线程已经准备好了，就等资源释放了     |

线程两种锁的模式：

| 模式      | 含义                           |
| :-------- | :----------------------------- |
| SHARED    | 表示线程以共享的模式等待锁     |
| EXCLUSIVE | 表示线程正在以独占的方式等待锁 |







### 独占锁加锁过程

![](/assets/imgs/image-20200904161535892.png)

![img](/assets/imgs/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2phdmFfbHl2ZWU=,size_16,color_FFFFFF,t_70.png)



#### **非公平锁模式：**

ReentrantLock的空参构造默认实现非公平锁

```java
// 非公平锁在加锁时先进行了一次CAS获取锁的尝试，如果获取到锁，直接执行，不需要排队阻塞
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

#### **公平锁模式：**

##### 线程t1进入：

当第一个线程t1 进入时，队头和队尾都是null，所以`h != t` 为False，return False，不需要排队。此时如果CAS操作成功，第3步的`tryAcquire()`返回true，第2步的`acquire(1)`未进入if，正常返回。***sync.lock()正常执行完毕。***  ***⚠️ 所以交替执行的线程不会使用到队列。***

```java
//1️⃣ Class FairSync
final void lock() {
    acquire(1);
}

//2️⃣ ---> acquire(1)来自于父类 AbstractQueuedSynchronizer
public final void acquire(int arg) {
  if (!tryAcquire(arg) && //⚠️  3️⃣带回来了true，所以无须对后半条语句进行判断，该函数正常返回咯
      acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}

//3️⃣ tryAcquire(arg) 尝试加锁，如果加锁失败则会调用acquireQueued方法加入队列去排队，如果加锁成功则不会调用
protected final boolean tryAcquire(int acquires) {
  final Thread current = Thread.currentThread(); // 获取当前线程
  int c = getState(); // Lock中要抢占的对象，类似于自实现锁中的 volatile int status=0;
  if (c == 0) {  // 0:自由状态/无人持有，可以抢占
    if (!hasQueuedPredecessors() &&  // 公平锁不会立刻进行CAS，还要判断自己是否需要排队 !hasQueuedPredecessors() 
        compareAndSetState(0, acquires)) { // 尝试 CAS 操作
      setExclusiveOwnerThread(current); // 如果CAS也改变成功，把当前线程设置为持锁线程
      return true; //返回2️⃣
    }
  }
  else if (current == getExclusiveOwnerThread()) {
    int nextc = c + acquires;
    if (nextc < 0)
      throw new Error("Maximum lock count exceeded");
    setState(nextc);
    return true;
  }
  return false;
}

// 4️⃣ hasQueuedPredecessors() 公平锁不会立刻进行CAS，还要判断自己是否需要排队 
public final boolean hasQueuedPredecessors() {
  // The correctness of this depends on head being initialized
  // before tail and on head.next being accurate if the current
  // thread is first in queue.
  Node t = tail; // Read fields in reverse initialization order
  Node h = head;
  Node s;
  return h != t &&
    ((s = h.next) == null || s.thread != Thread.currentThread());
}
```



##### 线程t2进入(未获得锁)：

![image-20200904221550944](/assets/imgs/image-20200904221550944-9230490.png)

t1线程以上锁，t2在t1未解锁时候访问同步代码；此时state已经被线程t1改为1，所以只能 return false。

> ⚠️ 一点小原则：AQS队列头部的thread永远为空！

![image-20200906101735855](/assets/imgs/image-20200906101735855.png)

![image-20200906132606830](/assets/imgs/image-20200906132606830.png)

**为什么要自旋两次，第一次ws=0,第二次ws=-1；**

ANSWER：多自旋转一次，为了尽量不让线程park()阻塞，此外ws=0是一个必要的存在，在shouldParkAfterFailedAcquire()方法中会对ws=0做一些逻辑判断，所以不能直接等于-1。（-1表示此线程处于睡眠状态）

```java
//1️⃣ Class FairSync
final void lock() {
    acquire(1);
}

//2️⃣ ---> acquire(1)来自于父类 AbstractQueuedSynchronizer
public final void acquire(int arg) {
  if (!tryAcquire(arg) && //  3️⃣带回来了false，继续判断后半条语句(是否需要入队)，->4️⃣
      acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) // 6️⃣
    selfInterrupt();
}

//3️⃣ tryAcquire(arg) 尝试加锁，如果加锁失败则会调用acquireQueued方法加入队列去排队，如果加锁成功则不会调用
protected final boolean tryAcquire(int acquires) {
  final Thread current = Thread.currentThread(); // 获取当前线程 t2
  int c = getState(); // Lock中要抢占的对象，类似于自实现锁中的 volatile int status=0;
  if (c == 0) {  // 此时c=1 执行else if
    if (!hasQueuedPredecessors() &&  
        compareAndSetState(0, acquires)) { 
      setExclusiveOwnerThread(current); 
      return true;
    }
  }
  else if (current == getExclusiveOwnerThread()) { // 判断当前线程是否和持锁线程相同，t2!=t1
    int nextc = c + acquires;
    if (nextc < 0)
      throw new Error("Maximum lock count exceeded");
    setState(nextc);
    return true;
  }
  return false; // 只能返回false咯，回到了2️⃣
}

// 4️⃣ addWaiter(Node.EXCLUSIVE), arg)
private Node addWaiter(Node mode) {
  Node node = new Node(Thread.currentThread(), mode); // 实例化一个node(双向链表)
  // Try the fast path of enq; backup to full enq on failure
  Node pred = tail; // pred = null;
  if (pred != null) { //false
    node.prev = pred;
    if (compareAndSetTail(pred, node)) {
      pred.next = node;
      return node;
    }
  }
  enq(node); // 到这咯->5️⃣
  return node; // 返回t2的node.->2️⃣
}

// 5️⃣ enq(node)
private Node enq(final Node node) {
  for (;;) { // 死循环
    Node t = tail; // t=null; //第二次循环t!=null咯，->else
    if (t == null) { // Must initialize
      if (compareAndSetHead(new Node())) // ⚠️ CAS 设置 AQS的头部(一个空的Node)！保证入队操作安全
        tail = head; // 尾部也指向这个空Node ⚠️ 注入灵魂: AQS队列头部的thread永远为空！
    } else { //第二次循环时进入
      node.prev = t; // 4️⃣中t2的node的prev指向空node，即入队
      if (compareAndSetTail(t, node)) { // AQS尾部是t就更改为node，原子操作
        t.next = node; // 空节点的next指向t2的node
        return t; // 返回队列头部节点t(空节点)->4️⃣
      }
    }
  }
}

// 6️⃣ acquireQueued()
final boolean acquireQueued(final Node node, int arg) {
  boolean failed = true; 
  try {
    boolean interrupted = false; // ReentrantLock 可以被打断
    for (;;) { // ⚠️ 死循环
      final Node p = node.predecessor();
      if (p == head && tryAcquire(arg)) { // ⚠️尝试CAS! 判断自己是不是第一个排队的(不是指头部)，是的话尝试获得锁 // 第二次循环 判断头部+再次自旋
        setHead(node);
        p.next = null; // help GC
        failed = false;
        return interrupted;
      }
      if (shouldParkAfterFailedAcquire(p, node) &&  // t2自旋失败来到这, -> 7️⃣ // 第二次循环前半部分为true，执行parkAndCheckInterrupt() -> 8️⃣
          parkAndCheckInterrupt())
        interrupted = true;
    }
  } finally {
    if (failed)
      cancelAcquire(node); // 失败 取消获得锁
  }
}

// 7️⃣ shouldParkAfterFailedAcquire()在获取锁失败后是否需要park
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
  int ws = pred.waitStatus; // ws=0 //第二次循环。 ws=-1
  if (ws == Node.SIGNAL) // Node.SIGNAL为-1
    return true; // 第二次循环 返回6️⃣true
  if (ws > 0) {
    do {
      node.prev = pred = pred.prev;
    } while (pred.waitStatus > 0);
    pred.next = node;
  } else {
    compareAndSetWaitStatus(pred, ws, Node.SIGNAL); // 把上一个节点null的 ws属性改为-1
  }
  return false; // 返回6️⃣
}

// 8️⃣
private final boolean parkAndCheckInterrupt() {
  LockSupport.park(this); // this是当前线程，阻塞t2
  return Thread.interrupted();
}
```



##### 线程t2进入(获得锁)：

![image-20200909151409617](/assets/imgs/image-20200909151409617.png)

```java
//1️⃣ Class FairSync
final void lock() {
    acquire(1);
}

//2️⃣ ---> acquire(1)来自于父类 AbstractQueuedSynchronizer
public final void acquire(int arg) {
  if (!tryAcquire(arg) && //  3️⃣(队列未初始化true，正常返回)
      acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) 
    selfInterrupt();
}

//3️⃣ tryAcquire(arg) 尝试加锁，如果加锁失败则会调用acquireQueued方法加入队列去排队，如果加锁成功则不会调用
protected final boolean tryAcquire(int acquires) {
  final Thread current = Thread.currentThread(); // 获取当前线程 t2
  int c = getState(); // Lock中要抢占的对象，类似于自实现锁中的 volatile int status=0;
  if (c == 0) {  // 此时c=0,进入循环
    if (!hasQueuedPredecessors() &&  // 判断t2是否需要排队->4️⃣（未初始化返回flase，继续判断）
        compareAndSetState(0, acquires)) { // CAS加锁
      setExclusiveOwnerThread(current); // 设置t2为持锁线程
      return true; //->2️⃣
    }
  }
  else if (current == getExclusiveOwnerThread()) { // 队列初始化完毕执行这段
    int nextc = c + acquires;
    if (nextc < 0)
      throw new Error("Maximum lock count exceeded");
    setState(nextc);
    return true;
  }
  return false; 
}

// 4️⃣ hasQueuedPredecessors() 
/**
* 下面提到的所有不需要排队，并不是字面意义，我实在想不出什么词语来描述这个“不需要排队”；不需要排队有两种情况
* 一：队列没有初始化，不需要排队，不需要排队，不需要排队；直接去加锁，但是可能会失败；为什么会失败呢？
* 假设两个线程同时来lock，都看到队列没有初始化，都认为不需要排队，都去进行CAS修改计数器；有一个必然失败
* 比如t1先拿到锁，那么另外一个t2则会CAS失败，这个时候t2就会去初始化队列，并排队
*
* 二：队列被初始化了，但是t2过来加锁，发觉队列当中第一个排队的就是自己；比如重入；
* 那么什么叫做第一个排队的呢？下面解释了，很重要往下看；
* 这个时候他也不需要排队，不需要排队，不需要排队；为什么不需要排队？
* 因为队列当中第一个排队的线程他会去尝试获取一下锁，因为有可能这个时候持有锁的那个线程可能释放了锁；
* 如果释放了就直接获取锁执行。但是如果没有释放他就会去排队，
* 所以这里的不需要排队，不是真的不需要排队
**/
public final boolean hasQueuedPredecessors() {
  // The correctness of this depends on head being initialized
  // before tail and on head.next being accurate if the current
  // thread is first in queue.
  Node t = tail; // Read fields in reverse initialization order
  Node h = head;
  Node s;
  /**
  * h != t 判断队首不等于队尾这里要分三种情况
  * 1、队列没有初始化，也就是第一个线程tf来加锁的时候那么这个时候队列没有初始化，
  * h和t都是null，那么这个时候判断不等于则不成立（false）那么由于是&&运算后面的就不会走了，
  * 直接返回false表示不需要排队，而前面又是取反（if (!hasQueuedPredecessors()），所以会直接去cas加锁。
  * 第一种情况总结：队列没有初始化没人排队，那么我直接不排队，直接上锁；合情合理、有理有据令人信服；
  *
  * 2、队列被初始化了，且队列元素>1,true，如果队列被初始化那么h!=t则成立；
  * 第二种情况总结，如果队列被初始化了，而且至少有一个人在排队那么自己也去排队；但是有个插曲；他会去看看那个第一个排队的人是不是自己，如果是自己那么他就去尝试加锁；尝试看看锁有没有释放
  *
  * 3、队列被初始化了，但是里面只有一个数据；什么情况下才会出现这种情况呢？ts加锁的时候里面就只有一个数据？
  * 其实不是，因为队列初始化的时候会虚拟一个h作为头结点，tc=ts作为第一个排队的节点；tf为持有锁的节点
  * 为什么这么做呢？因为AQS认为h永远是不排队的，假设你不虚拟节点出来那么ts就是h，
  *  而ts其实需要排队的，因为这个时候tf可能没有执行完，还持有着锁，ts得不到锁，故而他需要排队；
  * 那么为什么要虚拟为什么ts不直接排在tf之后呢，上面已经时说明白了，tf来上锁的时候队列都没有，他不进队列，
  * 故而ts无法排在tf之后，只能虚拟一个thread=null的节点出来（Node对象当中的thread为null）；
  * 那么问题来了；究竟什么时候会出现队列当中只有一个数据呢？假设原队列里面有5个人在排队，当前面4个都执行完了
  * 轮到第五个线程得到锁的时候；他会把自己设置成为头部，而尾部又没有，故而队列当中只有一个h就是第五个
  * 至于为什么需要把自己设置成头部；其实已经解释了，因为这个时候五个线程已经不排队了，他拿到锁了；
  * 所以他不参与排队，故而需要设置成为h；即头部；所以这个时间内，队列当中只有一个节点
  * 关于加锁成功后把自己设置成为头部的源码，后面会解析到；继续第三种情况的代码分析
  * 记得这个时候队列已经初始化了，但是只有一个数据，并且这个数据所代表的线程是持有锁
  * h != t false 由于后面是&&运算，故而返回false可以不参与运算，整个方法返回false；不需要排队
  * 第三种情况总结：如果队列当中只有一个节点，而这种情况我们分析了，这个节点就是当前持有锁的那个节点，故而我不需要排队，进行cas；尝试加锁。这是AQS的设计原理，他会判断你入队之前，队列里面有没有人排队；有没有人排队分两种情况；队列没有初始化，不需要排队；队列初始化了，按时只有一个节点，也是没人排队，自己先也不排队；只要认定自己不需要排队，则先尝试加锁；加锁失败之后再排队；
  */
  return h != t &&  // 未初始化 h=t=null,返回false；队列元素>1,true;队列元素=1
    ((s = h.next) == null || s.thread != Thread.currentThread()); //二号节点==null||二号节点的线程!=当前线程t2（是不是第一个排队的人来问，是的话就CAS加锁咯；在调用该方法的前提就是锁处于自由状态）
}

```



##### 线程t3进入(未获得锁)：

![image-20200906133113372](/assets/imgs/image-20200906133113372.png)

```java
//1️⃣ Class FairSync
final void lock() {
    acquire(1);
}

//2️⃣ ---> acquire(1)来自于父类 AbstractQueuedSynchronizer
public final void acquire(int arg) {
  if (!tryAcquire(arg) && //  3️⃣带回来了false，继续判断后半条语句(是否需要入队)，->4️⃣
      acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) // 5️⃣
    selfInterrupt();
}

//3️⃣ tryAcquire(arg) --> 来自于AbstractQueuedSynchronizer的实现类FairSync
protected final boolean tryAcquire(int acquires) {
  final Thread current = Thread.currentThread(); // 获取当前线程 t2
  int c = getState(); // Lock中要抢占的对象，类似于自实现锁中的 volatile int status=0;
  if (c == 0) {  // 此时c=1 执行else if
    if (!hasQueuedPredecessors() &&  
        compareAndSetState(0, acquires)) { 
      setExclusiveOwnerThread(current); 
      return true;
    }
  }
  else if (current == getExclusiveOwnerThread()) { // 判断当前线程是否和持锁线程相同，t2!=t1
    int nextc = c + acquires;
    if (nextc < 0)
      throw new Error("Maximum lock count exceeded");
    setState(nextc);
    return true;
  }
  return false; // 只能返回false咯，回到了2️⃣
}

// 4️⃣ addWaiter(Node.EXCLUSIVE), arg)
private Node addWaiter(Node mode) {
  Node node = new Node(Thread.currentThread(), mode); // 实例化一个t3的node(双向链表)
  // Try the fast path of enq; backup to full enq on failure
  Node pred = tail; // pred = t2 node;
  if (pred != null) { // true
    node.prev = pred; // t3的prev指向t2
    if (compareAndSetTail(pred, node)) { // CAS 尝试t3入队
      pred.next = node; // t2的next指向t2node
      return node; // 返回 2️⃣
    }
  }
  enq(node); 
  return node; 
}

// 5️⃣ acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
final boolean acquireQueued(final Node node, int arg) {
  boolean failed = true; 
  try {
    boolean interrupted = false; 
    for (;;) { // ⚠️ 死循环
      final Node p = node.predecessor(); // p = t2 node
      if (p == head && tryAcquire(arg)) {  // t2 node != head；不是第一个排队的就直接去乖乖排队吧
        setHead(node);
        p.next = null; // help GC
        failed = false;
        return interrupted;
      }
      if (shouldParkAfterFailedAcquire(p, node) &&  // 到这
          parkAndCheckInterrupt())
        interrupted = true;
    }
  } finally {
    if (failed)
      cancelAcquire(node); 
  }
}

// 6️⃣ shouldParkAfterFailedAcquire()在获取锁失败后是否需要park
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
  int ws = pred.waitStatus; //⚠️ ws拿的是前一个node t2的waitStatus！
  if (ws == Node.SIGNAL) // ws=0 执行else
    return true; 
  if (ws > 0) {
    do {
      node.prev = pred = pred.prev;
    } while (pred.waitStatus > 0);
    pred.next = node;
  } else { 
    compareAndSetWaitStatus(pred, ws, Node.SIGNAL); // 把t2的ws改为-1
  }
  return false; // 
}

// 7️⃣ 
private final boolean parkAndCheckInterrupt() {
  LockSupport.park(this); 
  return Thread.interrupted();
}
```

为什么不把自己的ws改为-1，而是修改上一个节点的ws。

ANSWER：因为阻塞了没办法修改自己；如果加在阻塞的上方，万一出现异常，阻塞就不会正常进行了，就不统一了。还有一个原因是当前线程ws=0在解锁过程有作用。

#### 加锁过程总结：

如果是第一个线程t1，那么和队列无关，线程直接持有锁。并且也不会初始化队列，如果接下来的线程都是交替执行，那么永远和AQS队列无关，都是直接线程持有锁，如果发生了竞争，比如t1持有锁的过程中T2来lock，那么这个时候就会初始化AQS，初始化AQS的时候会在队列的头部虚拟一个Thread为NULL的Node，因为队列当中的head永远是持有锁的那个node（除了第一次会虚拟一个，其他时候都是持有锁的那个线程锁封装的node），现在第一次的时候持有锁的是tf而tf不在队列当中所以虚拟了一个node节点，队列当中的除了head之外的所有的node都在park，当tf释放锁之后unpark某个（基本是队列当中的第二个，为什么是第二个呢？前面说过head永远是持有锁的那个node，当有时候也不会是第二个，比如第二个被cancel之后，至于为什么会被cancel，不在我们讨论范围之内，cancel的条件很苛刻，基本不会发生）node之后，node被唤醒，假设node是t2，那么这个时候会首先把t2变成head（sethead），在sethead方法里面会把t2代表的node设置为head，并且把node的Thread设置为null，为什么需要设置null？其实原因很简单，现在t2已经拿到锁了，node就不要排队了，那么node对Thread的引用就没有意义了。所以队列的head里面的Thread永远为null。

### 解锁过程

![image-20200909155215729](/assets/imgs/image-20200909155215729.png)

- 通过ReentrantLock的解锁方法Unlock进行解锁。
- Unlock会调用内部类Sync的Release方法，该方法继承于AQS。
- Release中会调用tryRelease方法，tryRelease需要自定义同步器实现，tryRelease只在ReentrantLock中的Sync实现，因此可以看出，释放锁的过程，并不区分是否为公平锁。
- 释放成功后，所有处理由AQS框架完成，与自定义同步器无关。



# ReentrantLock特性

ReentrantLock意思为可重入锁，指的是一个线程能够对一个临界资源重复加锁。为了帮助大家更好地理解ReentrantLock的特性，我们先将ReentrantLock跟常用的Synchronized进行比较，其特性如下：

|            | ReentrantLock                  | Synchronized     |
| ---------- | ------------------------------ | ---------------- |
| 锁实现机制 | 依赖AQS                        | 监视器模式       |
| 灵活性     | 支持响应中断、超时、尝试获取锁 | 不灵活           |
| 释放形式   | 必须显示调用unlock()释放锁     | 自动释放监视器   |
| 锁类型     | 公平锁&非公平锁                | 非公平锁         |
| 条件队列   | 可关联多个条件队列             | 关联一个条件队列 |
| 可重入性   | 可重入                         | 可重入           |





# [ReentrantLock打断](https://www.bilibili.com/video/BV19J411Q7R5?p=12)

> lock.lockInterruptibly();

例子：启动t1,t2线程，t1先获得锁并sleep，2s后t2还拿不到锁就打断它`t2.interrupt();`。

interrupt() 没有其他意义，只是一个标志。如果该线程有该打断标识，Thread.interrupted()==true(调用一次后恢复false)。此外如果要对打断进行处理，必须捕获异常后才能执行业务代码。

// lockInterruptibly --> parkAndCheckInterrupt --> 响应 --> 怎么才能响应？ --> 调用Thread.interrupted() 
// lock --> parkAndCheckInterrupt1(void) --> Thread.interrupted() 就不需要那么麻烦了

## t2调用lock()时：

```java
// 1️⃣
public final void acquire(int arg) {
  if (!tryAcquire(arg) && // 获取锁失败
      acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) // acquireQueued()也返回true
    selfInterrupt(); // 恢复用户行为，因为Thread.interrupted()==true(调用一次后恢复false)。
}

// 2️⃣
final boolean acquireQueued(final Node node, int arg) {
  boolean failed = true;
  try {
    boolean interrupted = false;
    for (;;) {
      final Node p = node.predecessor();
      if (p == head && tryAcquire(arg)) { // 假设第二次循环可以拿到锁了 设为头部维护链表...同上
        setHead(node);
        p.next = null; // help GC
        failed = false;
        return interrupted; // 返回true
      }
      /////与lockInterruptibly()的不同之处/////
      if (shouldParkAfterFailedAcquire(p, node) &&
          parkAndCheckInterrupt()) // 整个为true ⚠️所以这里其实并没有什么意义 仅仅是为了代码复用
        interrupted = true; // 被打断标识，为了方法的正常返回
    }
    /////与lockInterruptibly()的不同之处/////
  } finally {
    if (failed)
      cancelAcquire(node);
  }
}

// 3️⃣
private final boolean parkAndCheckInterrupt() {
  LockSupport.park(this);
  return Thread.interrupted(); // ⚠️ 这里调用了一次Thread.interrupted() 返回true且标识位还原
}

// 4️⃣
public static boolean interrupted() {
  return currentThread().isInterrupted(true);
}

// 当中断线程被唤醒时，并不知道被唤醒的原因，可能是当前线程在等待中被中断，也可能是释放了锁以后被唤醒。因此我们通过Thread.interrupted()方法检查中断标记（该方法返回了当前线程的中断状态，并将当前线程的中断标识设置为False），并记录下来，如果发现该线程被中断过，就再中断一次，还原用户状态。
static void selfInterrupt() {
  Thread.currentThread().interrupt();
}
```



## t2调用lockInterruptibly()时：

```java
// 1️⃣
public final void acquireInterruptibly(int arg) throws InterruptedException {
  if (Thread.interrupted())
    throw new InterruptedException(); // 判断在加锁之前有没有被打断，如果有就做出响应
  if (!tryAcquire(arg)) // 没被打断，正常执行。看能不能拿到锁
    doAcquireInterruptibly(arg); // 上锁失败，是否需要排队
}

// 2️⃣
private void doAcquireInterruptibly(int arg) throws InterruptedException {
  final Node node = addWaiter(Node.EXCLUSIVE);
  boolean failed = true;
  try {
    for (;;) {
      final Node p = node.predecessor();
      if (p == head && tryAcquire(arg)) {
        setHead(node);
        p.next = null; // help GC
        failed = false;
        return;
      }
      /////与lock()的不同之处/////
      if (shouldParkAfterFailedAcquire(p, node) &&
          parkAndCheckInterrupt()) // ⚠️  阻塞
        throw new InterruptedException(); // 醒来后发现被打断的话，变成了抛出异常
    }
    /////与lock()的不同之处/////
  } finally {
    if (failed)
      cancelAcquire(node);
  }
}

// 3️⃣ lock和lockInterruptibly都调用了这个方法(其实是老爷子偷懒了，不过复用性更好)
private final boolean parkAndCheckInterrupt() {
  LockSupport.park(this);  // 阻塞
  return Thread.interrupted(); // ⚠️ 醒来之前有被打断过的话为true，
}
```





# 重入锁ReentrantLock

**重入锁ReentrantLock**，顾名思义，就是支持重进入的锁，它表示该锁能够支持一个线程对资源的重复加锁。

可重入锁（也叫作递归锁），指的时同一线程外层函数获得锁之后，内层递归函数仍然能获取该锁的代码，在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁，也就是说，**线程可以进入任何一个它已经拥有的锁所同步着的代码块**。可重入锁最大的作用是**避免死锁**。

可重入锁主要解决了两个问题：

- 线程再次获取锁。锁需要去识别获取锁的线程是否为当前占据锁的线程，如果是，则再次成功获取。
- 锁的最终释放。线程重复n次获取了锁，随后在第n次释放该锁后，其他线程能够获取到该锁。锁的最终释放要求锁对于获取进行**计数自增**，计数表示当前锁被重复获取的次数，而锁被释放时，**计数自减**，当计数等于0时表示锁已经成功释放。

```java
//3️⃣ tryAcquire(arg) --> 来自于AbstractQueuedSynchronizer的实现类FairSync
protected final boolean tryAcquire(int acquires) {
  final Thread current = Thread.currentThread(); // 获取当前线程 t2
  int c = getState(); // Lock中要抢占的对象，类似于自实现锁中的 volatile int status=0;
  if (c == 0) {  // 此时c=1 执行else if
    if (!hasQueuedPredecessors() &&  
        compareAndSetState(0, acquires)) { 
      setExclusiveOwnerThread(current); 
      return true;
    }
  }
  // ⚠️⚠️⚠️ ReentrantLock 锁重入的实现！
  else if (current == getExclusiveOwnerThread()) { // 判断当前线程是否和持锁线程相同
    int nextc = c + acquires; // 如果是同一个线程，计数器变量+1；示当前锁被重复获取的次数
    if (nextc < 0)
      throw new Error("Maximum lock count exceeded");
    setState(nextc); // 设置锁的计数器
    return true;
  }
  return false;
}

// ReentrantLock.unlock() 调用了Sync中的tryRelease
protected final boolean tryRelease(int releases) {
  int c = getState() - releases; // 计数器 -1；锁被释放时，计数自减，当计数等于0时表示锁已经成功释放。
  if (Thread.currentThread() != getExclusiveOwnerThread())
    throw new IllegalMonitorStateException();
  boolean free = false;
  if (c == 0) {
    free = true;
    setExclusiveOwnerThread(null);
  }
  setState(c);
  return free;
}
```





# 读写锁

**写锁是独占的**，当写锁被获取到时，后续（非当前写操作线程）的读写操作都会被阻塞，写锁释放之后，所有操作继续执行。

**读锁是共享的**，多个线程可以共同读取数据。

一般情况下，读写锁的性能都会比排它锁好，因为大多数场景读是多于写的。在读多于写的情况下，读写锁能够提供比排它锁更好的并发性和吞吐量。Java并发包提供读写锁的实现是**ReentrantReadWriteLock**。

## **读写锁的实现原理：**

读写锁同样依赖自定义同步器来实现同步功能，而读写状态就是其同步器的同步状态。读写锁的自定义同步器需要在同步状态（一个整型变量）上维护多个读线程和一个写线程的状态。

如果在一个整型变量上维护多种状态，就一定需要**按位切割使用**这个变量，读写锁将变量切分成了两个部分，**高16位表示读，低16位表示写**，划分方式如图（当前同步状态表示一个线程已经获取了写锁，且重进入了两次，同时也连续获取了两次读锁）：

![image-20200903203931055](/assets/imgs/image-20200903203931055.png)

**写锁是一个支持重进入的排它锁**。如果当前线程已经获取了写锁，则增加写状态。如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者该线程不是已经获取写锁的线程，则当前线程进入等待状态。

**读锁是一个支持重进入的共享锁**，它能够被多个线程同时获取，在没有其他写线程访问（或者写状态为0）时，读锁总会被成功地获取，而所做的也只是（线程安全的）增加读状态。

