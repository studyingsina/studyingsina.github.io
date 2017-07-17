---
layout: post
title: "Thrift Mock Server Idea"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![未来](/css/pics/2017-book-list.jpg)

## 问题
当我们在写自动化case的时候，往往需要将外部的依赖mock掉，在我们公司，RPC组件用的Thrift，那么就涉及到一个问题，如果将Thrift给mock掉，并且我们这里是在做自动化测试（功能测试），不是单元测试，在单元测试的时候我们很容易mock。

## 思路

### Thrift原理
我们先来了解下Thrift一将调用的流程：

### Client
在Thrift Client端做一层代理，将Client的请求都打到一个Mock Server端口上，在代理层中解析Mock的数据；
优点：容易实现；
缺点：业务系统要引入Thrift Client的代理，对业务系统侵入；
总结：重Client、轻Server；

### Server
在Thrift Server端缓存Mock的数据，伪造成Thrift协议返回；
优点：业务系统无侵入；
缺点：实现难度要大一些；
总结：轻Client、重Server；

## 实现
理论上以上两种思路都可行，但是我个人更倾向于Server端这种，因为这种方式对业务系统无侵入，更接入于真实的测试环境，尽管这种方式我们要比Client多做一些工作；

所以我重点讲Server端的实现：

### Client

### Server


## 验证


