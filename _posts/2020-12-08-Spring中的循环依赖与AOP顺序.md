---
layout: post
title: "Spring中的循环依赖与AOP顺序"
date: 2020-12-08 12:43:16 +0800--
categories: [Java, ]
tags: [Java, spring, ]  

---

# Spring源码

## Spring源码方面的知识

- Spring bean的生命周期
- Spring 工厂，Spring容器，上下文 
- Spring BeanPostprocessor
- Spring 和 主流框架的源码
- Spring BeanFactory 和 FactoryBean的区别

## 谈谈你对Spring的理解

IOC、AOP只是作为Spring Framework里面一部分，同时还有还有events，resources，i18n，validation，data binding，type conversion，SpEL

![image-20200402092317669](/assets/imgs/image-20200402092317669.png)

## Spring上下文

从代码级别来说，就是指Spring Context

从源码级别，但我们初始化Spring Context的时候，一堆的Spring组件围绕在一起，使其能够正常工作，这个状态就被称为Spring环境

## Spring初始化

首先需要引入Spring的依赖，因为我们暂时只是初始化过程，只需要用到IOC

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.9.RELEASE</version>
</dependency>
/**
 * 配置类
 */
@Configuration
@ComponentScan("com.moxi.interview.study.spring")
public class AppConfig {
}


/**
 * Bean类
 */
@Component
public class BeanTest {
}
/**
 * Spring项目启动
 */
public class Test {
    public static void main(String[] args) {
        // 初始化
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(AppConfig.class);

        System.out.println(annotationConfigApplicationContext.getBean(BeanTest.class));
    }
}
https://github.com/spring-projects/spring-framework
// 方式1，目前用的比较多
AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(AppConfig.class)


invokeBeanFactoryPostProcessors() {
  // 扫描类：
  // 处理了各种import：例如@import("xxx.xml"), @MapperScanner, @CompoentScanner ..... 
}
/**
 * 扩展的BeanFactory
 */
public class TestBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

    }
}
```

在执行扫描的时候，它会扫描Spring 提供的 BeanFactoryPostProcessor，以及程序员扩展的

例如，很多需要对Spring进行扩展的，例如Mybatis，其实都是实现了 BeanFactoryPostProcessor接口



![image-20200402104306006](/assets/imgs/image-20200402104306006.png)

完整的加载图，左边红色部分就是Spring的加载过程，然后开放的原则，它还提供了很多扩展接口，让你可以干扰到Sring的加载过程，使得

![image-20200402104318234](/assets/imgs/image-20200402104318234.png)

- 最后我们将这个BeanDefinationMap放入了Spring单例池中

![image-20200402102958645](/assets/imgs/image-20200402102958645.png)



- 首先Spring会将全部的Class类，通过classLoader加载到JVM中

- 然后在通过扫描，创建很多BeanDefinition，我们通过反射将对应Class的信息填充到BeanDefinition中

  - 这里的BeanDefinition是用来描述Bean的，也就是Bean的一些信息存储

    ```
    RootBeanDefinition rootBeanDefinition = new RootBeanDefinition();
    rootBeanDefinition.setBeanClassName("BeanTest");
    rootBeanDefinition.setBeanClass(BeanTest.class);
    rootBeanDefinition.setScope("prototype");
    rootBeanDefinition.setLazyInit();
    ```

    

- 然后在把填充好的BeanDefinition一个个放入到Map中，Spring扫描了几个类，Map中就有几个类，这个Map被称为 BeanDefinitionMap

### Spring Bean实例化过程

![image-20200402101551621](/assets/imgs/image-20200402101551621.png)

普通类的实例化，就是通过javac编译成xxx.class文件，然后某一天通过new关键字进行实例化，JVM就会把这个class类加载到JVM内存中，这里面就涉及到了方法区，堆栈存储等。

### 普通类的实例化



同时在AnnotationConfigApplication方法的最后，还有一个refresh()方法，这个方法是整个Spring最核心的方法，这个方法的内部，同时调用了十多个方法，其中最重要的是 invokeBeanFactoryPostProcessors()

这个工厂最重要的功能就是产生Bean

![image-20200402100434794](/assets/imgs/image-20200402100434794.png)

调用AnnotationConfigApplicationContext无参构造方法，同时因为该类又继承了一个父类 GenericApplicationContext，子类在初始化的时候，还会调用父类的无参构造方法，在父类中，我们能够看到它初始化了一个BeanFactory，这就是我们经常提到的Spring工厂

![image-20200402100813001](/assets/imgs/image-20200402100813001.png)

同时在这个方法的内部，使用了this()

初始化Spring环境有两种方法，一种是通过注解的方式，一个是通过XML的方式

- 把类扫描出来（扫描出来后做了什么？）
- 把Bean实例化

Spring中的Bean不可能是直接new关键字创建出来的

## SpringBean的生命周期

tip：IDEA点击进去的源码目录，其实是IDEA反编译得到的，和原来的源码会存在一些出入，是IDEA专门优化过的，因此如果你需要修改源码的话，还是需要在官网下载对应的源码包

![image-20200402091642787](/assets/imgs/image-20200402091642787.png)

最后我们通过注解的方式，来获取Spring IOC扫描到的Bean，最后打印出来

3、Test.java，启动测试类

2、BeanTest.java，我们需要被扫描到的Bean

1、AppConfig.java，可以当成是扫描类，也就是配置我们需要扫描的目录

为了更加了解Spring初始化的过程，我们需要定义三个类



# Spring的AOP顺序

## Aop常用注解

- @Before：前置通知: 目标方法之前执行
- @After：后置通知: 目标方法之后执行（始终执行）
- @AfterReturning：返回后通知: 执行方法结束前执行(异常不执行)
- @AfterThrowing：异常通知: 出现异常时候执行
- @Around：环绕通知: 环绕目标方法执行



## 面试题

你肯定知道spring，那说说aop的全部通知顺序 springboot或springboot2对aop的执行顺序影响？

**ANSWER：**

springboot1底层是spring4；springboot2对应的则是spring5。

spring4默认用的是JDK的动态代理；spring5默认动态代理用的是cglib,不再是JDK的动态代理，因为JDK必须要实现接口，但有些类它并没有实现接口，所以更加通用的话就是cglib

Spring4：

- 正常情况下： **@Before(前置通知) ===> @After(后置通知) ===> @AfterReturning(正常返回)，环绕通知在@Before和业务逻辑之后。**

- 异常情况下：**@Before(前置通知) ===> @After(后置通知) ===> @AfterThrowing(方法异常)，出现异常了环绕通知只会出现一半，后面的BBB就没有了**

![image-20210105191011738](/assets/imgs/image-20210105191011738.png)

Spring5：

- 正常情况下： **@Before(前置通知) ===> @AfterReturning(正常返回) ===> @After(后置通知)，环绕通知在@Before和业务逻辑之后。**

- 异常情况下：**@Before(前置通知) ===> @AfterThrowing(方法异常) ===> @After(后置通知)，出现异常了环绕通知只会出现一半，后面的BBB就没有了**

![image-20210105195721886](/assets/imgs/image-20210105195721886.png)





# Spring循环依赖

## 大厂面试题

- 你解释下spring中的三级缓存?
- 三级缓存分别是什么?三个Map有什么异同?
- 什么是循环依赖?请你谈谈?看过spring源码吗?一般我们说的spring容器是什么 
- 如何检测是否籽在循环依赖?实际开发中见过循环依赖的异常吗?
- 多例的情况下，循环依赖问题为什么无法解决?
- 为什么一定要用3级缓存，1级/2级能不能行？
- 是否可以关闭循环依赖？

## 什么是循环依赖

多个bean之间相互依赖，形成了一个闭环。 比如:A依赖于B、B依赖于C、C依赖于A。代码如下：

```java
public class Test1{
  class A{
    B b;
  }
  
  class B{
    C c;
  }
  
  class C{
    A a;
  }
}
```

通常来说，如果问spring容器内部如何解决循环依赖， **一定是指默认的单例Bean中，属性互相引用的场景**。也就是说，Spring的循环依赖，是Spring容器注入时候出现的问题

![image-20210108190051744](/assets/imgs/image-20210108190051744.png)



## 两种注入方式对循环依赖的影响

> (官网说明)[https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-dependencies]

![image-20210108191858878](/assets/imgs/image-20210108191858878.png)

结论：我们AB循环依赖问题只要A的注入方式是setter且singleton, 就不会有循环依赖问题



## Spring容器循环依赖报错 🤔

### 循环依赖现象在Spring容器中注入依赖的对象，有2种情况

- 构造器方式注入依赖

  结论：构造器注入没有办法解决循环依赖， 你想让构造器注入支持循环依赖，是不存在的

  ```java
  // ServiceA
  @Component
  public class ServiceA {
  
      private ServiceB serviceB;
  
      public ServiceA(ServiceB serviceB){
          this.serviceB = serviceB;
      }
  }
  
  // ServiceB
  @Component
  public class ServiceB {
  
      private ServiceA serviceA;
  
      public ServiceB(ServiceA serviceA){
          this.serviceA = serviceA;
      }
  }
  
  // 构造器测试
  // ⚠️ 结论：构造器注入没有办法解决循环依赖， 你想让构造器注入支持循环依赖，是不存在的
  public class ClientConstructor {
      public static void main(String[] args) {
          new ServiceA(new ServiceB(...)); // 无限套娃
      }
  }
  ```

- 以set方式注入依赖

   结论：属性注入可以解决循环依赖

  ```java
  // ServiceAA
  @Component
  public class ServiceAA {
      private ServiceBB serviceBB;
  
      public void setServiceBB(ServiceBB serviceBB){
          this.serviceBB = serviceBB;
          System.out.println("A 里面设置了B");
      }
  }
  
  // ServiceBB
  @Component
  public class ServiceBB {
      private ServiceAA serviceAA;
  
      public void setServiceAA(ServiceAA serviceAA){
          this.serviceAA = serviceAA;
          System.out.println("B 里面设置了A");
      }
  }
  
  // 构造器测试
  // ⚠️ 结论：属性注入可以解决循环依赖
  public class ClientSet {
      public static void main(String[] args) {
          // 创建serviceAA
          ServiceAA a = new ServiceAA();
          // 创建serviceBB
          ServiceBB b = new ServiceBB();
          // 将serviceAA注入到serviceBB中
          b.setServiceAA(a);
          // 将serviceBB注入到serviceAA中
          a.setServiceBB(b);
      }
  }
  ```

  

### 重要code案例演示

1）code-java基础编码

```java
// A
public class A {
    private B b;

    public B getB() {
        return b;
    }

    public void setB(B b) {
        this.b = b;
    }

    public A(){
        System.out.println("---A created success");
    }
}
// B
public class B {
    private A a;

    public A getA() {
        return a;
    }

    public void setA(A a) {
        this.a = a;
    }

    public B(){
        System.out.println("---B created success");
    }
}

// ClientCode
public class ClientCode {
    public static void main(String[] args) {
        A a = new A();
        B b = new B();

        b.setA(a);
        a.setB(b);
    }
}
```





2）spring容器

- 默认的单例(**singleton**)的场景是**支持**循环依赖的，**不报错**
- 原型(**Prototype**)的场景是**不支持**循环依赖的，**报错**
  - 每次都new一个，说明没办法放到缓存里面，就用不了三级缓存。

步骤:

1. applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="a" class="cn.silince.circulardepend.importantdemo.A" scope="prototype">
        <property name="b" ref="b"/>
    </bean>

    <bean id="b" class="cn.silince.circulardepend.importantdemo.B" scope="prototype">
        <property name="a" ref="a"/>
    </bean>

</beans>
```

2. log4j.properties





3. 默认单例，修改为原型scope="prototype" 就导致了循环依赖错误



4. ClientSpringContainer



5. 循环依赖异常 --- `BeanCurrentlyInCreationException`

![image-20210108204608474](/assets/imgs/image-20210108204608474.png)





### 重要结论(spring内部通过3级缓存来解决循环依赖)

> ⚠️  **只有单例的bean会通过三级缓存提前暴露来解决循环依赖的问题**，而非单例的bean，每次从容器中获取都是一个新的对象，都会重新创建，所以非单例的bean是没有缓存的，不会将其放到三级缓存中。

关键类：DefaultSingletonBeanRegistry。

所谓的三级缓存其实就是spring容器内部用来解决循环依赖问题的三个map:

- 第一级缓存(也叫单例池)`singletonObjects`：存放已经经历了完整生命周期的Bean对象
- 第二级缓存`earlySingletonObjects`：存放早期暴露出来的Bean对象，Bean的生命周期未结束（属性还未填充完)
- 第三级缓存`Map<String,ObjectFactory<?>> singletonFactories`：存放可以生成Bean的工厂

![image-20210108205213065](/assets/imgs/image-20210108205213065.png)



# 循环依赖Debug 🤔

## 实例化/初初化

- 实例化：堆内存中申请一块内存空间(租赁好房子，自己的家具东西还没有搬家进去)
- 初始化：初始化属性填充，完成属性的各种赋值(装修、家电家具进场)

## 3大Map和四大方法

3大Map：

- 第一层singeltonObjects存放的是已经初始化好了的Bean
- 第二层earlySingletonObjects存放的是实例化了，但是未初始化的Bean
- 第三层singletonFactories存放的是FactoryBean。假如A类实现了FactoryBean，那么依赖注入的时候不是A类，而是A类产生的Bean

四大方法：

- getSingleton：希望从容器里面获得单例的bean，有的话直接拿，没有的话说明还未创建
- doCreateBean: 没有就创建bean
- populateBean: 创建完了以后，要填充属性
- addSingleton: 填充完了以后，再添加到容器进行使用

![image-20210108212739262](/assets/imgs/image-20210108212739262.png)

```java
/** 
 * 单例对象的缓存：bean 名称——bean实例，即所谓的单例池
 * 表示已经经历了完整生命周期的 Bean 对象
 * <b>第一级缓存</b>
 */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** 
 * 早期的单例对象的高速缓存：bean 名称——bean 实例。
 * 表示 Bean 的生命周期还没走完(Bean 的属性还未填充)就把这个 Bean 存入该缓存中
 * 也就是实例化但是未初始化的 bean 放入该缓存里
 * <b>第二级缓存</b>
 */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

/** 
 * 单例工厂的高速缓存：bean 名称——ObjectFactory。
 * 表示存放生成 bean 的工厂
 * <b>第三级缓存</b>
 */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```



## A/B两对象在三级缓存中的迁移说明

- **A创建过程中需要B，于是A将自己放到三级缓存里面，去实例化B**
- **B实例化的时候发现需要A，于是B先查一级缓存，没有，再查二级缓存，还是没有，再查三级缓存，找到了A，然后把三级缓存里面的这个A放到二级缓存里面，并删除三级缓存里面的A** 

- **B顺利初始化完毕，将自己放到一级缓存里面(此时B里面的A依然是创建中状态)，然后回来接着创建A，此时B已经创建结束，直接从一级缓存里面拿到B，然后完成创建，并将A自己放到一级缓存里面。**

断点与流程：

![image-20210110180206375](/assets/imgs/image-20210110180206375.png)

![image-20210110180754571](/assets/imgs/image-20210110180754571.png)



# 总结 

![image-20210110195934990](/assets/imgs/image-20210110195934990.png)

![image-20210110200040114](/assets/imgs/image-20210110200040114.png)

![image-20210110200054293](/assets/imgs/image-20210110200054293.png)



  