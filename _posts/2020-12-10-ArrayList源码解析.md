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

![image-20210117122612602](/assets/imgs/image-20210117122612602.png)



> 数组的长度是有限制的，而ArrayList是可以存放任意数量对象，长度不受限制，那么他是怎么实现的呢？

其实实现方式比较简单，他就是通过数组扩容的方式去实现的。

就比如我们现在有一个长度为10的数组，现在我们要新增一个元素，发现已经满了，那ArrayList会怎么做呢？

![image-20210117123339404](/assets/imgs/image-20210117123339404.png)

第一步他会重新定义一个长度为**10+10/2**的数组也就是新增一个长度为15的数组。

![image-20210117123353718](/assets/imgs/image-20210117123353718.png)

然后把原数组的数据，原封不动的复制到新数组中，这个时候再把指向原数组的地址换到新数组，ArrayList就这样完成了一次改头换面。

![image-20210117123406279](/assets/imgs/image-20210117123406279.png)

⚠️：因为我们在使用ArrayList的时候一般不会设置初始值的大小，**ArrayList默认的大小就刚好是10。**

![image-20210117123435341](/assets/imgs/image-20210117123435341.png)

然后你们也可以看到，他的构造方法，如果你传入了初始值大小，那就使用你传入的参数，如果没，那就使用默认的，一切都是有迹可循的。

> ArrayList的默认数组大小为什么是10？

据说是因为sun的程序员对一系列广泛使用的程序代码进行了调研，结果就是10这个长度的数组是最常用的最有效率的。也有说就是随便起的一个数字，8个12个都没什么区别，只是因为10这个数组比较的圆满而已。



> 我记得你说到了，他增删很慢，你能说一下ArrayList在增删的时候是怎么做的么？主要说一下他为啥慢。

他有指定index新增，也有直接新增的，**在这之前他会有一步校验长度的判断ensureCapacityInternal**，就是说如果长度不够，是需要扩容的。

![image-20210117135305807](/assets/imgs/image-20210117135305807.png)

在扩容的时候，老版本的jdk和8以后的版本是有区别的，8之后的效率更高了，**采用了位运算，右移一位，其实就是除以2这个操作。**

![image-20210117135444504](/assets/imgs/image-20210117135444504.png)

指定位置新增的时候，在校验之后的操作很简单，就是数组的copy，大家可以看下代码：

![image-20210117135536999](/assets/imgs/image-20210117135536999.png)

不知道大家看懂**arraycopy**的代码没有，我画个图解释下，你可能就明白一点：

比如有下面这样一个数组我需要在index 5的位置去新增一个元素A

![Image](/assets/imgs/640-20210117140405307.png)

那从代码里面我们可以看到，他复制了一个数组，是从index 5的位置开始的，然后把它放在了index 5+1的位置

![Image](/assets/imgs/640-20210117140405335.png)

给我们要新增的元素腾出了位置，然后在index的位置放入元素A就完成了新增的操作了

![Image](/assets/imgs/640-20210117140405349.png)

至于为啥说他效率低，我想我不说你也应该知道了，我这只是在一个这么小的List里面操作，要是我去一个几百几千几万大小的List新增一个元素，**那就需要后面所有的元素都复制**，然后如果再涉及到扩容啥的就更慢了不是嘛。



> ⭐️ 真实的场景题：ArrayList(int initialCapacity) 会不会初始化数组大小？

<u>***不会初始化数组大小！***</u>

而且将构造函数与`initialCapacity`结合使用，然后使用set()会抛出异常，尽管该数组已创建，但是大小设置不正确。

使用`sureCapacity()`也不起作用，因为它基于`elementData`数组而不是大小。

还有其他副作用，这是因为带有`sureCapacity()`的静`态DEFAULT_CAPACITY`。

进行此工作的唯一方法是在使用构造函数后，根据需要使用`add()`多次。

大家可能有点懵，我直接操作一下代码，大家会发现我们虽然对`ArrayList`设置了初始大小，但是我们打印List大小的时候还是0，我们操作下标set值的时候也会报错，数组下标越界。

再结合源码，大家仔细品读一下，这是Java Bug里面的一个经典问题了，还是很有意思的，大家平时可能也不会注意这个点。

![image-20210117141049908](/assets/imgs/image-20210117141049908.png)



> ArrayList插入删除一定慢么？

取决于你删除的元素离数组末端有多远，ArrayList拿来作为堆栈来用还是挺合适的，push和pop操作完全不涉及数据移动操作。



> 那他的删除怎么实现的呢？

删除其实跟新增是一样的，不过叫是叫删除，但是在代码里面我们发现，他还是在copy一个数组。



> 为啥是copy数组呢？

![Image](/assets/imgs/640-20210117142859986.png)

继续打个比方，我们现在要删除下面这个数组中的index5这个位置

![Image](/assets/imgs/640-20210117142859957.png)

那代码他就复制一个index5+1开始到最后的数组，然后把它放到index开始的位置

![Image](/assets/imgs/640-20210117142859966.png)

index5的位置就成功被”删除“了其实就是被覆盖了，给了你被删除的感觉。

同理他的效率也低，因为数组如果很大的话，一样需要复制和移动的位置就大了。



> ArrayList是线程安全的么？

当然不是，线程安全版本的数组容器是Vector。

Vector的实现很简单，就是把所有的方法统统加上synchronized就完事了。

你也可以不使用Vector，用`Collections.synchronizedList`把一个普通ArrayList包装成一个线程安全版本的数组容器也可以，原理同Vector是一样的，就是给所有的方法套上一层synchronized。



> ArrayList用来做队列合适么？

队列一般是FIFO（先入先出）的，如果用ArrayList做队列，就需要在数组尾部追加数据，数组头部删除数组，反过来也可以。

**但是无论如何总会有一个操作会涉及到数组的数据搬迁，这个是比较耗费性能的。**

**结论：ArrayList不适合做队列。**



> 那数组适合用来做队列么？

**数组是非常合适的。**

比如`ArrayBlockingQueue`内部实现就是一个环形队列，它是一个定长队列，内部是用一个定长数组来实现的。

另外著名的Disruptor开源Library也是用环形数组来实现的超高性能队列，具体原理不做解释，比较复杂。

**简单点说就是使用两个偏移量来标记数组的读位置和写位置，如果超过长度就折回到数组开头，前提是它们是定长数组。**



以数组作为底层数据结构时，一般讲队列实现为循环队列。这是因为队列在顺序存储上的不足：每次从数组头部删除元素（出队）后，需要将头部以后的所有元素往前移动一个位置，这是一个时间复杂度为O（n）的操作：

![image-20210117144600622](/assets/imgs/image-20210117144600622.png)

可能有人说，把队首标志往后移动不就不用移动元素了吗？的确，但那样会造成数组空间的“流失”。

我们希望队列的插入与删除操作都是O(1)的时间复杂度，同时不会造成数组空间的浪费，我们应该使用**循环队列**。

所谓的循环队列，可以把数组看出一个首尾相连的圆环，删除元素时将队首标志往后移动，**添加元素时若数组尾部已经没有空间，则考虑数组头部的空间是否空闲**，如果是，则在数组头部进行插入。

![image-20210117144626985](/assets/imgs/image-20210117144626985.png)

那么我们如何判断队列是空队列还是已满呢？

1. **栈空： 队首标志=队尾标志时，表示栈空**，即红绿两个标志在图中重叠时为栈空。
2. **栈满 : 队尾+1 = 队首时，表示栈空**。图三最下面的队列即为一个满队列。尽管还有一个空位，我们不存储元素。





> ArrayList的遍历和LinkedList遍历性能比较如何？

**论遍历ArrayList要比LinkedList快得多，ArrayList遍历最大的优势在于内存的连续性，**CPU的内部缓存结构会缓存连续的内存片段，可以大幅降低读取内存的性能开销。





# ArrayList常用的方法总结

- `boolean add(E e)` 将指定的元素添加到此列表的尾部。

- `void add(int index, E element)` 将指定的元素插入此列表中的指定位置。

- `boolean addAll(Collection c) `按照指定 collection 的迭代器所返回的元素顺序，将该 collection 中的所有元素添加到此列表的尾部。

- `boolean addAll(int index, Collection c)` 从指定的位置开始，将指定 collection 中的所有元素插入到此列表中。

- `void clear()` 移除此列表中的所有元素。

- `Object clone()` 返回此 ArrayList 实例的浅表副本。

- `boolean contains(Object o)` 如果此列表中包含指定的元素，则返回 true。

- `void ensureCapacity(int minCapacity)` 如有必要，增加此 ArrayList 实例的容量，以确保它至少能够容纳最小容量参数所指定的元素数。

- `E get(int index) `返回此列表中指定位置上的元素。

- `int indexOf(Object o) `返回此列表中首次出现的指定元素的索引，或如果此列表不包含元素，则返回 -1。

- `boolean isEmpty() `如果此列表中没有元素，则返回 true

- `int lastIndexOf(Object o) `返回此列表中最后一次出现的指定元素的索引，或如果此列表不包含索引，则返回 -1。

- `E remove(int index)` 移除此列表中指定位置上的元素。

- `boolean remove(Object o) `移除此列表中首次出现的指定元素（如果存在）。

- `protected void removeRange(int fromIndex, int toIndex)` 移除列表中索引在 fromIndex（包括）和 toIndex（不包括）之间的所有元素。

- `E set(int index, E element) `用指定的元素替代此列表中指定位置上的元素。

- `int size()` 返回此列表中的元素数。

- `Object[] toArray()` 按适当顺序（从第一个到最后一个元素）返回包含此列表中所有元素的数组。

- `T[] toArray(T[] a)` 按适当顺序（从第一个到最后一个元素）返回包含此列表中所有元素的数组；返回数组的运行时类型是指定数组的运行时类型。

- `void trimToSize() `将此 ArrayList 实例的容量调整为列表的当前大小。



# ArrayList源码解析

## 类图

![image-20210117145422636](/assets/imgs/image-20210117145422636.png)

- 实现了`RandomAccess`接口，可以随机访问
- 实现了`Cloneable`接口，可以克隆
- 实现了`Serializable`接口，可以序列化、反序列化
- 实现了`List`接口，是`List`的实现类之一
- 实现了`Collection`接口，是`Java Collections Framework`成员之一
- 实现了`Iterable`接口，可以使用`for-each`迭代



## 属性

1. 为什么空实例默认数组有的时候是`EMPTY_ELEMENTDATA`，而又有的时候是`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`
2. 为什么`elementData`要被`transient`修饰
3. 为什么`elementData`没有被`private`修饰？难道正如注释所写的**non-private to simplify nested class access**

```java
// 序列化版本UID
private static final long
        serialVersionUID = 8683452581122892189L;

/**
 * 默认的初始容量
 */
private static final int
        DEFAULT_CAPACITY = 10;

/**
 * 用于空实例的共享空数组实例
 * new ArrayList(0);
 */
private static final Object[]
        EMPTY_ELEMENTDATA = {};

/**
 * 用于提供默认大小的实例的共享空数组实例
 * new ArrayList();
 */
private static final Object[]
        DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 存储ArrayList元素的数组缓冲区
 * ArrayList的容量，是数组的长度
 * 
 * non-private to simplify nested class access
 */
transient Object[] elementData;

/**
 * ArrayList中元素的数量
 */
private int size;
```



## 构造方法

### 带初始容量的构造方法

- 如果`initialCapacity < 0`，就创建一个新的长度是`initialCapacity`的数组
- 如果`initialCapacity == 0`，就使用EMPTY_ELEMENTDATA
- 其他情况，`initialCapacity`不合法，抛出异常

```java
/**
 * 带一个初始容量参数的构造方法
 *
 * @param  initialCapacity  初始容量
 * @throws  如果初始容量非法就抛出
 *          IllegalArgumentException
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData =
                new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException(
                "Illegal Capacity: "+ initialCapacity);
    }
}
```



### 无参构造方法

```java
/**
 * 无参构造方法 将elementData 赋值为
 *   DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 */
public ArrayList() {
    this.elementData =
            DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```



### 带一个集合参数的构造方法

- 使用将集合转换为数组的方法
- 为了防止`c.toArray()`方法不正确的执行，导致没有返回`Object[]`，特殊做了处理
- 如果数组大小等于`0`，则使用 `EMPTY_ELEMENTDATA`

```java
/**
 * 带一个集合参数的构造方法
 *
 * @param c 集合，代表集合中的元素会被放到list中
 * @throws 如果集合为空，抛出NullPointerException
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    // 如果 size != 0
    if ((size = elementData.length) != 0) {
        // c.toArray 可能不正确的，不返回 Object[]
        // https://bugs.openjdk.java.net/browse/JDK-6260652
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(
                    elementData, size, Object[].class);
    } else {
        // size == 0
        // 将EMPTY_ELEMENTDATA 赋值给 elementData
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

那么问题来了，什么情况下`c.toArray()`会不返回`Object[]`呢？

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>(Arrays.asList("list"));
    // class java.util.ArrayList
    System.out.println(list.getClass());

    Object[] listArray = list.toArray();
    // class [Ljava.lang.Object;
    System.out.println(listArray.getClass());
    listArray[0] = new Object();

    System.out.println();

    List<String> asList = Arrays.asList("asList");
    // class java.util.Arrays$ArrayList
    System.out.println(asList.getClass());

    Object[] asListArray = asList.toArray();
    // class [Ljava.lang.String;
    System.out.println(asListArray.getClass());
    // java.lang.ArrayStoreException
    asListArray[0] = new Object();
}
```

我们通过这个例子可以看出来，`java.util.ArrayList.toArray()`方法会返回`Object[]`没有问题。

而`java.util.Arrays`的私有内部类ArrayList的`toArray()`方法可能不返回`Object[]`。

***<u>原因：</u>***

我们看ArrayList的`toArray()`方法源码：

```java
public Object[] toArray() {
    // ArrayLisy中 elementData是这样定义的
    // transient Object[] elementData;
    return Arrays.copyOf(elementData, size);
}
```

使用了`Arrays.copyOf()`方法：

```java
public static <T> T[] copyOf(T[] original, int newLength) {
    // original.getClass() 是 class [Ljava.lang.Object
    return (T[]) copyOf(original, newLength, original.getClass());
}
```

`copyOf()`的具体实现：

```java
public static <T,U> T[] copyOf(U[] original, 
          int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    /**
     * 如果newType是Object[] copy 数组 类型就是 Object 
     * 否则就是 newType 类型
     */
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

我们知道ArrayList中`elementData`就是`Object[]`类型，所以ArrayList的`toArray()`方法必然会返回`Object[]`。

我们再看一下`java.util.Arrays`的内部ArrayList源码（截取的部分源码）：

```java
private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable {

    // 存储元素的数组
    private final E[] a;

    ArrayList(E[] array) {
        // 直接把接收的数组 赋值 给 a
        a = Objects.requireNonNull(array);
    }

    /**
     * obj 为空抛出异常
     * 不为空 返回 obj
     */
    public static <T> T requireNonNull(T obj) {
        if (obj == null)
            throw new NullPointerException();
        return obj;
    }

    @Override
    public Object[] toArray() {
        // 返回 a 的克隆对象
        return a.clone();
    }

}
```

这是`Arrays.asList()`方法源码

```java
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```

不难看出来`java.util.Arrays`的内部ArrayList的`toArray()`方法，是构造方法接收什么类型的数组，就返回什么类型的数组。

所以，在我们上面的例子中，实际上返回的是String类型的数组，再将其中的元素赋值成`Object`类型的，自然报错。





## 插入方法

### 在列表最后添加指定元素

### 在指定位置添加指定元素

### 插入方法调用的其他私有方法



## 扩容方法

## 移除方法

### 移除指定元素方法

### 私有移除方法



## 查找方法

### 查找指定元素的所在位置

### 查找指定位置的元素



## 序列化方法

## 反序列化方法

## 创建子数组

## 迭代器

### 创建迭代器方法

### Itr属性

### Itr的hasNext() 方法

### Itr的next()方法

### Itr的remove()方法



## ArrayList类注释翻译