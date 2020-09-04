---
layout: post
title:  "从ReentrantLock的实现看AQS的原理"
date:   2020-09-03 18:58:06 +0800--
categories: [Java]
tags: [高并发, ]  
---

[Java中的锁及AQS实现原理](https://blog.csdn.net/TJtulong/article/details/105345940)

[美团 从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)

[Java AQS底层原理解析](https://segmentfault.com/a/1190000020521611?utm_source=sf-related)



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



# 队列同步器AQS

***队列同步器AbstractQueuedSynchronizer（AQS）是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态***，通过内置的**FIFO队列**来完成资源获取线程的排队工作，并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。

> ReentrantLock的lock方法调用的是Sync的lock()，而Sync继承于AbstractQueuedSynchronizer。

## AQS的实现

### FIFO队列

同步器依赖内部的同步队列（一个FIFO双向队列）来完成同步状态的管理，当前线程获取同步状态失败时，同步器会将当前线程以及等待状态等信息构造成为一个节点（Node）并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。





# 重入锁ReentrantLock

**重入锁ReentrantLock**，顾名思义，就是支持重进入的锁，它表示该锁能够支持一个线程对资源的重复加锁。

可重入锁（也叫作递归锁），指的时同一线程外层函数获得锁之后，内层递归函数仍然能获取该锁的代码，在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁，也就是说，**线程可以进入任何一个它已经拥有的锁所同步着的代码块**。可重入锁最大的作用是**避免死锁**。

可重入锁主要解决了两个问题：

- 线程再次获取锁。锁需要去识别获取锁的线程是否为当前占据锁的线程，如果是，则再次成功获取。
- 锁的最终释放。线程重复n次获取了锁，随后在第n次释放该锁后，其他线程能够获取到该锁。锁的最终释放要求锁对于获取进行**计数自增**，计数表示当前锁被重复获取的次数，而锁被释放时，**计数自减**，当计数等于0时表示锁已经成功释放。

# 公平与非公平锁

ReentrantLock还可以实现非公平锁，其区别在于非公平锁在加锁时先进行了一次CAS获取锁的尝试，如果获取到锁，直接执行，不需要排队阻塞。

刚释放锁的线程再次获取同步状态的几率会非常大，使得其他线程只能在同步队列中等待。

**公平性锁保证了锁的获取按照FIFO原则，而代价是进行大量的线程切换。非公平性锁虽然可能造成线程“饥饿”，但极少的线程切换，保证了其更大的吞吐量。**

# 读写锁

**写锁是独占的**，当写锁被获取到时，后续（非当前写操作线程）的读写操作都会被阻塞，写锁释放之后，所有操作继续执行。

**读锁是共享的**，多个线程可以共同读取数据。

一般情况下，读写锁的性能都会比排它锁好，因为大多数场景读是多于写的。在读多于写的情况下，读写锁能够提供比排它锁更好的并发性和吞吐量。Java并发包提供读写锁的实现是**ReentrantReadWriteLock**。

## **读写锁的实现原理：**

读写锁同样依赖自定义同步器来实现同步功能，而读写状态就是其同步器的同步状态。读写锁的自定义同步器需要在同步状态（一个整型变量）上维护多个读线程和一个写线程的状态。

如果在一个整型变量上维护多种状态，就一定需要**按位切割使用**这个变量，读写锁将变量切分成了两个部分，**高16位表示读，低16位表示写**，划分方式如图（当前同步状态表示一个线程已经获取了写锁，且重进入了两次，同时也连续获取了两次读锁）：

![image-20200903203931055](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20200903203931055.png)

**写锁是一个支持重进入的排它锁**。如果当前线程已经获取了写锁，则增加写状态。如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者该线程不是已经获取写锁的线程，则当前线程进入等待状态。

**读锁是一个支持重进入的共享锁**，它能够被多个线程同时获取，在没有其他写线程访问（或者写状态为0）时，读锁总会被成功地获取，而所做的也只是（线程安全的）增加读状态。

