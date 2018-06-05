---
layout: post
title: "Java多线程系列"
description: "Java 多线程 Multi Thread"
categories: 
- Tech
tags:
- Tech 多线程
---

* content
{:toc}

![Metrics](/css/pics/2017-11-13-multi-thread.jpg)

# 背景
温故知新、梳理自己的知识点；

业务开发过程中自己写多线程的场景其实并不是太多，但是又不能不了解多线程，我们整个编程环境就是在多线程下，一不小心，便会出现线程不安全的代码；

本系列文章重在讲解Java中如何应用多线程、先讲应用不讲原理，对：就是科普；

# 目录

以写一个Java版的WebServer的例子，去学习如何应用多线程编程中的一些知识点；

当然，这其中也会涉及到一些其它知识点，比如：Java Socket、重构、面向对象设计等；

本系列文章的正确打开方式：先将完整代码下载运行之、再亲手撸一遍、再查相关知识点、再后看每篇文章内容；

看完本系列：希望你知道如何构建一个简单的WebServer、学会应用多线程编程、知道业务上的增删改查是由哪些线程完成的；

## 为什么需要多线程

因为一个人的力量有限、群众的力量更大；

## 多线程编程面临的问题

1. 线程互斥与同步
2. 线程通信

理解这两个问题的关键点在于：共享变量；

## 实例学习多线程

### 先写一个简单WebServer
[《Java多线程系列-先写一个简单WebServer》](http://www.longtask.net/2017/11/20/a-simple-webserver/)

### 开始给主线程减压
[《Java多线程系列-开始给主线程减压》](http://www.longtask.net/2017/11/22/blocking-queue/)

### 开始给工作线程减压
[《Java多线程系列-开始给工作线程减压》](http://www.longtask.net/2017/11/21/reduce-worker/)

### 缓解忙等问题
[《Java多线程系列-缓解忙等问题》](http://www.longtask.net/2017/11/21/reduce-worker/)

### 增加工作线程-线程池
[《Java多线程系列-增加工作线程-线程池》](http://www.longtask.net/2017/11/23/thread-pool/)

### JCU API实现
[《Java多线程系列-JCU-API实现》](http://www.longtask.net/2017/11/24/jcu/)

## 知识点

Java中多线程编程提供的解决方案

1. JDK1.5前的实现，即：synchronized、wait、notify等机制；
2. JCU的实现，即：lock、condition等机制；； 

## 工作中的一些案例

业务开发过程中，也遇到过一些案例，比如死锁、线程不安全的代码，不过得慢慢找找。

# 总结

我不是一个聪明的人，至少我刚学编程的时候，对多线程编程这块理解的很慢，抓不住问题的核心点，以至于做了一些无用功，当时特别希望有个师傅、学长之类的人能指导一下、如果能手把手教就更好了；

写的这个小Demo希望可以给初学者一些帮助，当然也顺便梳理下自己的知识点；

如果后续有时间，会将这一系列录制成一个视频...；

如果再有时间，会再写其它系列文章：比如讲些线程底层原理、JVM实战等等；

# 引用

1. [《How Tomcat Works》](https://book.douban.com/subject/1943128/)
2. [《Java Thread Programming》](https://book.douban.com/subject/1864049/)
3. [《Tomcat源码》](http://tomcat.apache.org/)
4. [《JCU源码》](http://www.java.com/)
