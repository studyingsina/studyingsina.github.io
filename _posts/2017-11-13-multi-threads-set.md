---
layout: post
title: "Java多线程系列"
description: "Java 多线程 Multi Thread"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![Metrics](/css/pics/2017-11-13-multi-thread.jpg)

# 背景
温故知新、梳理自己的知识点；

业务开发过程中自己写多线程的场景其实并不是太多，但是又不能不了解多线程，我们整个编程环境就是在多线程下，一不小心，便会出现线程不安全的代码；

本系列文章重在讲解Java中如何应用多线程、先讲应用不讲原理；

# 目录

本系列文章，以写一个Java版的WebServer的例子，去学习如何应用多线程编程中的一些知识点；

当然，这其中也会涉及到一些其它知识点，比如：Java Socket、重构、面向对象设计等；

## 为什么需要多线程

## 多线程编程面临的问题

### 线程互斥与同步

### 线程通信

## 实例学习多线程

### 先写一个简单WebServer
[《Java多线程系列-先写一个简单WebServer》](http://www.longtask.net/2017/11/20/a-simple-webserver/)

### 开始给主线程减压
[《Java多线程系列-开始给主线程减压》](http://www.longtask.net/2017/11/22/blocking-queue/)

### 开始给工作线程减压
[《Java多线程系列-开始给工作线程减压》](http://www.longtask.net/2017/11/21/reduce-worker/)

### 缓解忙等问题

[《Java多线程系列-缓解忙等问题》](http://www.longtask.net/2017/11/21/reduce-worker/)

### WebServer简单重构

### JCU API实现

## 一些技巧

### Java中多线程编程提供的解决方案

### 多线程编程的一些经验

### 线程池

## 工作中的一些案例

# 总结

# 引用

[《How Tomcat Works》](https://book.douban.com/subject/1943128/)

[《Java Thread Programming》](https://book.douban.com/subject/1864049/)

[《Tomcat源码》](http://tomcat.apache.org/)
