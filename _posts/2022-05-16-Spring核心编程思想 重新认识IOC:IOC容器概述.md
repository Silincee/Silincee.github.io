---
layout: post
title:  "Spring核心编程思想 重新认识IOC/IOC容器概述"
date:   2022-05-16 22:55:06 +0800--
categories: [Spring, ]
tags: [框架, ]  

---

# 依赖查找

> 依赖查找可以分为按 名称、类型、注解查找三种(包括单个类型和集合类型)

## 根据Bean名称/类型查找

```java
/**
* 根据类型查找
*/
private static void lookupByType(BeanFactory beanFactory) {
  User user = beanFactory.getBean(User.class);
  System.out.println("根据类型查找: " + user);
}

/**
* 根据名称查找
*/
private static void lookupByName(BeanFactory beanFactory) {
  User user = (User) beanFactory.getBean("user");
  System.out.println("根据名称查找: " + user);
}
```



## 根据Bean类型/类型查找集合对象

```java
/**
* 按照类型查找集合对象
*/
private static void lookupCollectionByType(BeanFactory beanFactory) {
  if (beanFactory instanceof ListableBeanFactory) {
    Map<String, User> userList = ((ListableBeanFactory) beanFactory).getBeansOfType(User.class);
    System.out.println("查找到的所有类型为User的集合对象: " + userList);
  }
}
```



## 根据Java注解查找

```java
/**
* 根据注解查找集合对象
*/
private static void lookupCollectionByAnnotationType(BeanFactory beanFactory) {
  if (beanFactory instanceof ListableBeanFactory) {
    ListableBeanFactory listableBeanFactory = ((ListableBeanFactory) beanFactory);
    Map<String, User> userList = (Map) listableBeanFactory.getBeansWithAnnotation(Root.class);
    System.out.println("查找到的所有标注 @Root 的User的集合对象: " + userList);
  }
}
```



## 实时/延迟查找

首先明确一下什么是延迟查找，一般来说通过`@Autowired`注解注入一个具体对象的方式是属于实时依赖查找，注入的前提是要保证对象已经被创建。而使用延迟查找的方式是我可以不注入对象的本身，而是通过注入一个代理对象，在需要用到的地方再去取其中真实的对象来使用 ，`ObjectFactory`提供的就是这样一种能力。

- 实时查找：马上查找，马上索得
- 延迟查找：并不是指延迟加载Bean。而是不注入对象的本身，而是通过注入一个代理对象，在需要用到的地方再去取其中真实的对象来使用 ，`ObjectFactory`提供的就是这样一种能力。

```java
/**
  * 实时查找
  */
private static void lookupInRealTime(BeanFactory beanFactory) {
  User user = (User) beanFactory.getBean("user");
  System.out.println("实时查找: " + user);
}

/**
  * 延时查找
  */
private static void lookupInLazy(BeanFactory beanFactory) {
  ObjectFactory<User> objectFactory = (ObjectFactory<User>) beanFactory.getBean("objectFactory");
  User user = objectFactory.getObject();
  System.out.println("延迟查找: " + user);
}
```





# 依赖注入

1）根据Bean名称注入

2）根据Bean类型注入(单个Bean对象/集合Bean对象)

```xml
<bean id="userRepository" class="cn.rin.thinking.ioc.overview.repository.UserRepository"
      autowire="byType"> <!--  Auto-Wiring 自动绑定-->
  <!-- 硬编码,手动配置-->
  <!--        <property name="users">-->
  <!--            <util:list>-->
  <!--                <ref bean="rootUser"/>-->
  <!--                <ref bean="user"/>-->
  <!--            </util:list>-->
  <!--        </property>-->
</bean>
```

3）注入容器内建Bean对象

4）注入非Bean对象

5）注入类型(实时注入/延时注入)

```java
/**
 * 用户信息仓库
 *
 * @author rin
 * @since 2023.03.19
 */
@Data
public class UserRepository {

  /** 
    * 定义 bean 
    */ 
  private Collection<User> users;

  /**
    * 内建非 Bean对象(依赖)
    */
  private BeanFactory beanFactory;

  /** 
    * 延时注入
    */ 
  private ObjectFactory<User> userObjectFactory;
}
```





# 依赖来源

1）自定义Bean，如业务Bean

2）容器内建Bean对象，如org.springframework.core.env.Environment

3）容器内建依赖，如BeanFactory

# 配置元信息

1）Bean定义配置

- 基于XML文件
- 基于Properties文件
- 基于Java注解
- 基于Java API

2）Ioc容器配置

- 基于XML文件
- 基于Java注解
- 基于Java API

3）外部化属性配置

- 基于Java注解(@Value)

# IoC容器





# Spring应用上下文



# IoC容器生命周期
