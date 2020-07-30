---
layout: post
title:  "SpringMVC ResponseEntity<T>"
date:   2020-05-01 10:08:06 +0800--
categories: [Java,]
tags: [Spring, JavaWeb,  ]  
---

在传统的开发过程中，我们的控制CONTROLLER层通常需要转向一个JSP视图；但随着WEB2.0相关技术的崛起，我们很多时候只需要返回数据即可，而不是一个JSP页面。

- ResponseEntity：表示整个HTTP响应：状态代码，标题和正文。因此，我们可以使用它来完全配置HTTP响应，它是一个对象。

- @ResponseBody：返回json格式的结果

- @ResponseStatus：返回状态

  

ResponseEntity 表示整个HTTP响应：状态代码，标题和正文。因此，我们可以使用它来完全配置HTTP响应，它是一个对象，而*@ResponseBody*和@ResponseStatus是注解，适合于简单直接的场合。



## **ResponseEntity**

> 官方API：[][][ResponseEntity API](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/ResponseEntity.html)

**ResponseEntity**是一种泛型类型。因此，我们可以使用任何类型作为响应主体

### 将 “Hello World!” 作为字符串返回给REST端

```java
@Controller
public class XXXController{
 @GetMapping("/hello")
 public ResponseEntity<String> hello() {
   // public static <T> ResponseEntity<T> ok(T body)
   // 一种捷径去创建ResponseEntity，通过被给予的body和设置了OK的状态
   return ResponseEntity.ok("Hello World!");
 }
```

### 设置自定义标头

```java
@GetMapping("/customHeader")
ResponseEntity<String> customHeader() {
   return ResponseEntity.ok()
         .header("Custom-Header", "foo")
         .body("Custom header set")
```

### 如果将一个对象放入,返回对象的json列表

```java
@GetMapping("/hello")
public ResponseEntity<String> hello() {
   return ResponseEntity.ok(new User(‘jdon’));
 }
```

