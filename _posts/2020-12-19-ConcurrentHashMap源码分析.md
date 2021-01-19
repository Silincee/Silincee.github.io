---
layout: post
title:  "ConcurrentHashMap源码分析"
date:   2020-12-19 13:16:06 +0800--
categories: [Java, 高并发]
tags: [HashMap, 高并发, 集合框架, ]  
---

# 多线程下的 HashMap 存在的问题







# 为什么选用 ConcurrentHashMap







# ConcurrentHashMap 1.7 源码解析

## 底层存储结构

## 常用变量

## 构造函数

## put() 方法

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