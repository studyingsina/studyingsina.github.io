---
layout: post
title: "Spring Aop Deadlock Case"
description: "Java Spring Aop Deadlock Case"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![deadlock](/css/pics/2017-09-12-deadlock.jpg)

## 问题
隔壁组线上环境有一次发布，6台机器，5台发布成功，1台出现死锁，jstack打印出线程栈，当然为解决线上问题，我们立马对此应用进行重启、再次重启就没有死锁现象了；

用的Spring版本为3.1.2.RELEASE;

### 疑问1
为何同样的部署环境、同样的业务代码，6台机器，只有1台会出现死锁？

### 疑问2
为何出现死锁的机器，重启之后又没有死锁了么？

## 原因
通过jstack的线程栈分析，是两个线程，每个线程都要去争两把锁，分别叫锁1和锁2，线程1获得了锁1、等待获取锁2，线程2获得了锁2、等待获取锁1，互相等待对方释放资源，于是很经典的死锁发生的；

当然，此处我分析完jstack后进行了一些简化，实际的jstack更复杂，因为代码层层嵌套，获取的锁不止两个、且栈中存在公司代码不便贴出，故我将复杂例子简单化描述了；随后我会写一个Demo模拟这种死锁情况；

再贴一个官方Bug链接：[SPR-14241](https://jira.spring.io/browse/SPR-14241)

### 重现问题
在分析了死锁原因后，我写了一个Demo，可以通过IDE Debug去重现这个死锁问题：[AopDeadlockTester.java](https://github.com/studyingsina/spring_use/blob/master/src/test/java/com/studying/aop/AopDeadlockTester.java)

## 解决
其实查到问题原因后，想解决方法就容易的多了，当然、最官方还是得看上边贴的链接：[SPR-14241](https://jira.spring.io/browse/SPR-14241)

## 问题

### Spring Aop 过程是怎样的


### Spring 创建Bean的过程是怎样的


### 生成代理的方式有哪些

