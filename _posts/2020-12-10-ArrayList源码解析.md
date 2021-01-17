---
layout: post
title: "ArrayList源码解析"
date: 2020-12-10 11:21:16 +0800--
categories: [Java, ]
tags: [Java, ArrayList, ]  

---

# ArrayList的常见问题

> ArrayList是什么，可以用来干嘛？

ArrayList就是数组列表，主要用来装载数据，当我们装载的是基本类型的数据int，long，boolean，short，byte…的时候我们只能存储他们对应的包装类，它的主要底层实现是数组`Object[] elementData`。

与它类似的是LinkedList，和LinkedList相比，**它的查找和访问元素的速度较快，但新增，删除的速度较慢。**

**小结**：ArrayList底层是用数组实现的存储。

**特点**：查询效率高，增删效率低，线程不安全。使用频率很高。



> 为啥线程 不安全还使用它呢？

因为我们正常使用的场景中，都是用来查询，不会涉及太频繁的增删，如果涉及频繁的增删，可以使用LinkedList，如果你需要线程安全就使用Vector，这就是三者的区别了，实际开发过程中还是ArrayList使用最多的。

不存在一个集合工具是查询效率又高，增删效率也高的，还线程安全的，至于为啥大家看代码就知道了，因为数据结构的特性就是优劣共存的，想找个平衡点很难，牺牲了性能，那就安全，牺牲了安全那就快速。

⚠️：这里还是强调下大家**不要为了用而用**，我记得我以前最开始工作就有这个毛病。就是不管三七二十一，就是为了用而用，别人用我也用，也不管他的性能啊，是否线程安全啥的，所幸最开始公司接触的，都是线下的系统，而且没啥并发。



> 您说它的底层实现是数组，但是数组的大小是定长的，如果我们不断的往里面添加数据的话，不会有问题吗？

ArrayList可以通过构造方法在初始化的时候指定底层数组的大小。

通过无参构造方法的方式`ArrayList()`初始化，则赋值底层数`Object[] elementData`为一个默认空数组

`Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}`所以数组容量为0，只有真正对数据进行添加`add`时，才分配默认`DEFAULT_CAPACITY = 10` 的初始容量。

大家可以分别看下他的无参构造器和有参构造器，无参就是默认大小，有参会判断参数。

![image-20210117122612602](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210117122612602.png)



> 数组的长度是有限制的，而ArrayList是可以存放任意数量对象，长度不受限制，那么他是怎么实现的呢？

其实实现方式比较简单，他就是通过数组扩容的方式去实现的。

就比如我们现在有一个长度为10的数组，现在我们要新增一个元素，发现已经满了，那ArrayList会怎么做呢？

![image-20210117123339404](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210117123339404.png)

第一步他会重新定义一个长度为**10+10/2**的数组也就是新增一个长度为15的数组。

![image-20210117123353718](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210117123353718.png)

然后把原数组的数据，原封不动的复制到新数组中，这个时候再把指向原数组的地址换到新数组，ArrayList就这样完成了一次改头换面。

![image-20210117123406279](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210117123406279.png)

⚠️：因为我们在使用ArrayList的时候一般不会设置初始值的大小，**ArrayList默认的大小就刚好是10。**

![image-20210117123435341](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210117123435341.png)

然后你们也可以看到，他的构造方法，如果你传入了初始值大小，那就使用你传入的参数，如果没，那就使用默认的，一切都是有迹可循的。

> ArrayList的默认数组大小为什么是10？

据说是因为sun的程序员对一系列广泛使用的程序代码进行了调研，结果就是10这个长度的数组是最常用的最有效率的。也有说就是随便起的一个数字，8个12个都没什么区别，只是因为10这个数组比较的圆满而已。



> 我记得你说到了，他增删很慢，你能说一下ArrayList在增删的时候是怎么做的么？主要说一下他为啥慢。

他有指定index新增，也有直接新增的，**在这之前他会有一步校验长度的判断ensureCapacityInternal**，就是说如果长度不够，是需要扩容的。

![image-20210117135305807](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210117135305807.png)

在扩容的时候，老版本的jdk和8以后的版本是有区别的，8之后的效率更高了，**采用了位运算，右移一位，其实就是除以2这个操作。**

![image-20210117135444504](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210117135444504.png)

指定位置新增的时候，在校验之后的操作很简单，就是数组的copy，大家可以看下代码：

![image-20210117135536999](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210117135536999.png)

不知道大家看懂**arraycopy**的代码没有，我画个图解释下，你可能就明白一点：

比如有下面这样一个数组我需要在index 5的位置去新增一个元素A

![Image](/Users/silince/Develop/博客/blog_to_git/assets/imgs/640-20210117140405307.png)

那从代码里面我们可以看到，他复制了一个数组，是从index 5的位置开始的，然后把它放在了index 5+1的位置

![Image](/Users/silince/Develop/博客/blog_to_git/assets/imgs/640-20210117140405335.png)

给我们要新增的元素腾出了位置，然后在index的位置放入元素A就完成了新增的操作了

![Image](/Users/silince/Develop/博客/blog_to_git/assets/imgs/640-20210117140405349.png)

至于为啥说他效率低，我想我不说你也应该知道了，我这只是在一个这么小的List里面操作，要是我去一个几百几千几万大小的List新增一个元素，**那就需要后面所有的元素都复制**，然后如果再涉及到扩容啥的就更慢了不是嘛。



> ⭐️ 真实的场景题：ArrayList(int initialCapacity) 会不会初始化数组大小？

<u>***不会初始化数组大小！***</u>

而且将构造函数与`initialCapacity`结合使用，然后使用set()会抛出异常，尽管该数组已创建，但是大小设置不正确。

使用`sureCapacity()`也不起作用，因为它基于`elementData`数组而不是大小。

还有其他副作用，这是因为带有`sureCapacity()`的静`态DEFAULT_CAPACITY`。

进行此工作的唯一方法是在使用构造函数后，根据需要使用`add()`多次。

大家可能有点懵，我直接操作一下代码，大家会发现我们虽然对`ArrayList`设置了初始大小，但是我们打印List大小的时候还是0，我们操作下标set值的时候也会报错，数组下标越界。

再结合源码，大家仔细品读一下，这是Java Bug里面的一个经典问题了，还是很有意思的，大家平时可能也不会注意这个点。

![image-20210117141049908](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20210117141049908.png)



> ArrayList插入删除一定慢么？

取决于你删除的元素离数组末端有多远，ArrayList拿来作为堆栈来用还是挺合适的，push和pop操作完全不涉及数据移动操作。



> 那他的删除怎么实现的呢？

删除其实跟新增是一样的，不过叫是叫删除，但是在代码里面我们发现，他还是在copy一个数组。



> 为啥是copy数组呢？

![Image](/Users/silince/Develop/博客/blog_to_git/assets/imgs/640-20210117142859986.png)

继续打个比方，我们现在要删除下面这个数组中的index5这个位置

![Image](/Users/silince/Develop/博客/blog_to_git/assets/imgs/640-20210117142859957.png)

那代码他就复制一个index5+1开始到最后的数组，然后把它放到index开始的位置

![Image](/Users/silince/Develop/博客/blog_to_git/assets/imgs/640-20210117142859966.png)

index5的位置就成功被”删除“了其实就是被覆盖了，给了你被删除的感觉。

同理他的效率也低，因为数组如果很大的话，一样需要复制和移动的位置就大了。



> ArrayList是线程安全的么？

当然不是，线程安全版本的数组容器是Vector。

Vector的实现很简单，就是把所有的方法统统加上synchronized就完事了。

你也可以不使用Vector，用Collections.synchronizedList把一个普通ArrayList包装成一个线程安全版本的数组容器也可以，原理同Vector是一样的，就是给所有的方法套上一层synchronized。



> ArrayList用来做队列合适么？

队列一般是FIFO（先入先出）的，如果用ArrayList做队列，就需要在数组尾部追加数据，数组头部删除数组，反过来也可以。

但是无论如何总会有一个操作会涉及到数组的数据搬迁，这个是比较耗费性能的。

**结论：ArrayList不适合做队列。**



> 那数组适合用来做队列么？

这个女人是魔鬼么？不过还是得微笑面对！