---
layout: post
title: "Thrift Mock Server - 2"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![Mock](/css/pics/2017-07-17-jewellery.jpg)

## 问题
继上一篇[Thrift Mock Server](http://www.longtask.net/2017/07/17/thrift-mock-server/)，有同学问了，Thrift也支持[注解](https://github.com/facebook/swift)方式开发，不用每次都得写IDL文件，生成Thrift的一堆类文件，我实际项目中用的注解，如何开发Mock Server？

## IDL方式

我们先看下用IDL开发的流程，如图：

![Thrift-IDL-Process](/css/pics/2017-07-23-thrift-idl-process.png)

1. 编写IDL文件；
2. 将IDL文件编译成目标代码；
3. 实现IDL文件中定义的服务方法；
4. 发布Thrift服务；
5. 服务提供方将目标代码打包供客户端引入、调用；

Thrift为了简化Java开发，支持纯Java的实现，用几个简单的注解便可将上述1~2两步给省掉；

当然了，有得有失、恒古不变，省掉的两步不是白省的，得靠其它地方做很多工作来弥补；

## 思路

### 注解方式调用流程
先分析问题，分析清楚了，便知道如何处理；

我们先来了解下Thrift注解调用的流程，如图：

![thrift-anno-invoke](/css/pics/2017-07-23-thrift-anno-invoke.png)

其实这张图跟[Thrift Mock Server](http://www.longtask.net/2017/07/17/thrift-mock-server/)的图类似，只是在目标代码这块不一样，即图中黄色两块：Codec-Client、Codec-Server，相比原来IDL的方式，我们其实还是需要目标代码，只是说目标代码不再由我们自己去生成，而是通过[Facebook swift](https://github.com/facebook/swift)动态生成：

1. 读取注解标注的服务；
2. 用Codec编/解码动态生成Class文件；
3. JVM载入这些动态生成的Class文件；
4. 剩余的步骤跟IDL方式一样；

看到这个过程，其实如何Mock注解方式，我们便知道了，原来IDL方式的Mock方式，注解完全可以使用；

唯一不同的便是我们得根据注解方便去实现一遍，主要工作量在看懂swift api、生成每个服务的Codec；

我也实现了一个注解的[demo](https://github.com/studyingsina/spring_use):

先运行ThriftServerV2Demo.java，再运行ThriftClientV2Demo.java，便可看到结果；

此处只是拿原生的Thrift写了一个实现，离实际正式环境使用还有一定差距，不过思路是一致的，比如我们公司对原生Thrift做了一层封装，加入了服务自动注册、调用链追踪等功能，即支持IDL方式、也支持注解方式，我便按上边的思路实现了一个Mock Server。

## 总结
用注解方式我们提高了开发效率，但是引了新的问题，大家可以思考下，用注解方式又有哪些不如IDL的方面呢？
