---
layout: post
title:  "Mybatis笔记"
date:   2020-03-13 12:48:06 +0800--
categories: [JavaWeb]
tags: [Java, Mybatis, ]  
---

## Mybatis框架概述

​	mybatis是一个基于java的持久成框架，内部封装了jdbc，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程，使用了ORM思想实现了结果集的封装。

​	mybatis通过xml或注解的方式把将要执行的各种statement配置起来，并通过java对象和statement中的sql的动态参数进行映射生成最终执行的sql语句，最后有mybatis框架执行sql并将结果映射为java对象并返回。

​	**ORM**：Object Relational Mapping 对象关系映射（把数据库表和实体类及实体类的属性对应起来，让我们可以操作实体类就实现操作数据库表）



## Mybatis的入门

### mybatis的环境搭建

1. 创建maven工程并导入坐标

2. 创建实体类和dao的接口

3. 创建mybatis的主配置文件 `SqlMapConfig.xml`

4. 创建映射配置文件 `UserDao.xml`

### 环境搭建的注意事项

1. 创建UserDao.xml和UserDao.java时名称是为了和之前的知识保持一致。在Mybatis中它把持久成的操作接口名称和映射文件叫做Mapper，所以UserDao和UserMapper是一样的。

2. 在Idea中创建目录的时候，它和包是不一样的。

   包在创建时： cn.silince.dao 是三级结构

   目录在创建时： cn.silince.dao 是一级目录

3.   mybatis的映射配置文件位置必须和dao接口的包结构相同
4. 映射配置文件的mapper标签namespace属性的值必须是dao接口的全限定类名
5. 映射配置文件的操作配置（select），id属性的值必须是dao接口的方法名

> 当我们遵从了3，4，5点之后，我们在开发中就无须再写dao的实现类

### 入门案例

```java
public class MybatisTest {
    public static void main(String[] args) throws IOException {
        //1 读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2 创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        //3 使用工厂生产一个SqlSession对象
        SqlSession session = factory.openSession();
        //4 使用SqlSession创建Dao接口的代理对象
        UserDao userDao=session.getMapper(UserDao.class);
        //5 使用代理对象执行dao中的方法
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
        }
        //6 释放资源
        session.close();
        in.close();
    }
}
```

![入门案例的分析](/assets/imgs/入门案例的分析.png)

#### Ps

​		不要忘记在映射配置中告知mybatis要封装到哪个实体类中

​		配置方式：指定实体类的全限定类名    `<select id="findAll" resultType="cn.itcast.domain.User">`

#### ⭐️mybatis基于注解的入门案例：

​		把UserDao.xml移除，在dao接口的方法上使用@Select注解，并指定sql语句，同时需要在SqlMapConfig.xml中的mapper配置时候，使用class属性指定到dao接口的全限定类名。

​		在实际开发中，都是越简便越好，所以都是采用不写dao实现类的方式，不管使用xml还是注解配置，但是mybatis他是支持写dao实现类的。

```java
UserDao:
public interface UserDao {
    @Select("select * from user")
    List<User> findAll();
}
SqlMapConfig.xml:
<!--    指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件
  如果是使用注解来配置的话，此处应该使用class属性指定被注解的dao全限定类名
  -->
    <mappers>
        <mapper class="cn.itcast.dao.UserDao"/>
    </mappers>
```



#### 自定义mybatis分析

![自定义Mybatis分析](/assets/imgs/自定义Mybatis分析.png)



