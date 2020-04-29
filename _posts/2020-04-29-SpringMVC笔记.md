---
layout: post
title:  "springMVC笔记"
date:   2020-04-29 20:31:06 +0800--
categories: [JavaEE]
tags: [Java, spring, ]  

---

## 三层架构和MVC

1. 三层架构
   1. 开发服务器端程序，一般都基于两种形式，一种C/S架构程序，一种B/S架构程序 
   2. 使用Java语言基本上都是开发B/S架构的程序，B/S架构又分成了三层架构 
   3. 三层架构
      - 表现层:WEB层，用来和客户端进行数据交互的。表现层一般会采用MVC的设计模型 
      - 业务层:处理公司具体的业务逻辑的
      - 持久层:用来操作数据库的

2. MVC模型
   1. MVC 全名是Model View Controller 模型视图控制器，每个部分各司其职。
   2. Model：数据模型，JavaBean的类，用来进行数据封装。
   3. View：指JSP、HTML用来展示数据给用户
   4. Controller：用来接收用户的请求，整个流程的控制器。用来进行数据校验等。

## SpringMVC的入门案例

1. 需求

   ![image-20200429210647020](/assets/imgs/image-20200429210647020.png)

2. 入门案例的执行流程

   1. 当启动Tomcat服务器的时候，因为配置了load-on-startup标签，所以会创建DispatcherServlet对象， 就会加载springmvc.xml配置文件
   2. 开启了注解扫描，那么HelloController对象就会被创建
   3. 从index.jsp发送请求，请求会先到达DispatcherServlet核心控制器，根据配置@RequestMapping注解

   找到执行的具体方法

   4. 根据执行方法的返回值，再根据配置的视图解析器，去指定的目录下查找指定名称的JSP文件

   5. Tomcat服务器渲染页面，做出响应

![image-20200429220240757](/assets/imgs/image-20200429220240757.png)

3. SpringMVC官方提供图形

4. 入门案例中的组件分析
   1. 前端控制器(DispatcherServlet) 
   2. 处理器映射器(HandlerMapping) 
   3. 处理器(Handler)
   4. 处理器适配器(HandlAdapter)
   5. 视图解析器(View Resolver)
   6. 视图(View)

## RequestMapping注解

1. `@RequestMapping`注解的作用是建立请求URL和处理方法之间的对应关系 
2.  RequestMapping注解可以作用在方法和类上
   1. 作用在类上:第一级的访问目录
   2. 作用在方法上:第二级的访问目录
   3. 细节:路径可以不编写 / 表示应用的根目录开始
   4. 细节:`${ pageContext.request.contextPath }`也可以省略不写，但是路径上不能写 `/`
3. RequestMapping的属性
   - path/value：指定请求路径的url
   - method：指定该方法的请求方式
   - params：指定限制请求参数的条件
   - headers：发送请求中必须包含的请求头