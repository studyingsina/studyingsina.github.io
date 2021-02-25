---
layout: post
title: "一次http请求在Tomcat中的执行流程"
categories: 
- Life
tags:
- Life
---

* content
{:toc}

![A-HTTP-Request-In-The-Tomcat](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.001.jpeg)

## 背景

组内要求每人都要进行分享，故准备一个组内的小分享，因为实际工作中使用过Tomcat和Jetty作为Servlet容器，所以先拿Tomcat来开个头；

通过了解一次HTTP请求在Tomcat中的流转来认识Tomcat内的重要组件及设计；

![tomcat002](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.002.jpeg)

## 需求

假设时间回到20年前，你是[Sun Microsystems](https://en.wikipedia.org/wiki/Sun_Microsystems)的一名开发人员，产品经理[James Duncan Davidson](https://en.wikipedia.org/wiki/James_Duncan_Davidson)给你提了一个需求：

需求背景：为了推广Sun的Servlet规范，让广大程序员喜欢用Servlet开发应用、降低程序员的开发成本，需要提供一个开源的Servlet容器；

具体需求如下：

1. 支持Servlet+JSP规范
2. 支持WebServer
3. 开源
4. 易扩展
5. 高性能
6. 自定义/被替换

## Demo

先写一个Demo，看实现一个WebServer+Servlet容器需要做哪些最基本的事情；

![tomcat004](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.004.jpeg)

我们来简化下Demo版需要做的逻辑：

1. 启动服务器监听端口,如8080
2. 开始监听客户端请求,如http请求
3. 解析http请求参数
4. 业务自己逻辑处理,如查询DB并返回数据
5. 将返回数据按http协议格式返回

在Tomcat中也存在这5步，只不过Tomcat考虑的逻辑更多更周全些；

## 功能拆分

先把变和不变的逻辑拆分开来，哪些是易变的逻辑、哪些是不易变的逻辑？

第1、2、3、5步骤是不易变的，基本上每一个服务器都要做的事情；

第4步是每一个提供服务的应用都不一样的；

![tomcat011](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.011.jpeg)

先把第4步放一放，对1、2、3、5步骤再进行拆分：第1步启动服务器监听端口没什么可说的，new一个ServerSocket，3和5步骤是对HTTP请求/输出流的解析和封装，我们将3、5放在一起：

![tomcat012](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.012.jpeg)

## 性能

从线程的角度来看，目前只有一个应用启动线程（main线程）在处理监听端口-解析http请求的事情，这样当请求多的话、main线程的处理效率成为瓶颈，所以我们将监听端口和请求处理分开：

![tomcat013](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.013.jpeg)

一共三类线程，main线程、监听线程、工作线程池，处理模型是这样的：

1. main线程启动ServerSocket，并创建一个监听线程（Acceptor）+一个HTTP任务处理线程线池（每个HTTP请求一个线程）
2. 监听线程只用来监听Socket，收到Socket后放入线程池的队列中（Queue）
3. 每一个工作线程处理各自的HTTP请求（Socket）

## Servlet

上图的Controller其实就对应Servlet，来看下Servlet这部分：

![tomcat014](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.014.jpeg)

以HTTP请求流转的视角将上图简化一下：

![tomcat015](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.015.jpeg)

一个Servlet可以认为处理一个或一类HTTP请求，通常部署一个服务应用会存在不止一个Servlet，所以我们要开发的Servlet容器需要支持多个Servlet的情况；（Wrapper）

多个Servlet如果认为是一个应用的话，那么有些时候我们希望一个服务器能够部署不止一个应用；（Context）

多个应用如果认为是一个应用集的话，那么有些时候我们希望一个服务器能够部署不止一个应用集；（Host）

多个应用集如果认为是一个服务引擎的话，那么我们先打住；（Engine）

Tomcat对于Servlet支持层级的抽象、分为了4层：

![tomcat016](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.016.jpeg)

简化下上边的图

![tomcat017](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.017.jpeg)

当浏览器发出一个请求过来，Tomcat先交给Engine--->Engine先判断要交给哪个Host--->Host再判断要交给哪个Context--->Context再判断要交给哪个Servlet处理；

打个比方，XX组织要求公司的应用支持IPV6标准，找到公司CEO，CEO判断这个事该CTO负责（而不是CFO、或其它的CXO），CTO接收到这个需求判断应该由基础网络工程部处理（而不是其它技术部门），层层分解、并且每层将任务派发下去时会附近一些指示等等；

既然每一层（Engine-->Host-->Context-->ServletWrapper）将请求往下分派时都会对请求做一部分加工处理，那么Tomcat是如何处理这种场景呢？

没错，就是责任链模式，上图：

![tomcat018](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.018.jpeg)

Pipeline是责任链、责任链中可以配置多个过滤器-Vavle，不过Tomcat的责任链做了一个变化，每个责任链中至少要配置一个基础的过滤器，这个过滤器承担的职责是将请求传递到下一层级的Pipeline中，请求流是这样流转的：

request-->Pipeline(Engine的责任链)-->自定义各种过滤器（Engine的）-->EngineVavle（Engine的基础过滤器）-->Pipeline（Host的责任链）-->自定义各种过滤器（Host的）-->HostVavle（Host的基础过滤器）-->Pipeline（Context的责任链）-->自定义各种过滤器（Context的）-->ContextVavle（Context的基础过滤器）-->Pipeline（ServletWrapper的责任链）-->自定义各种过滤器（Context的）-->WrapperVavle（Wrapper的）-->Servlet

这4层每一层都有一个责任链，然后4个责任链串到一起来将请求传递到Servlet，看图：

![tomcat019](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.019.jpeg)

当最后的WrapperVavle拿到请求后，要把请求传递给Servlet，但是我们都知道，Servlet规范是支持在请求前也配置过滤器的，其实是就Sun在制定Servlet规范时，在应用层给了广大程序员提供可扩展，要求Servlet窗口再实现一个应用层的责任链（过滤器链）

## tomcat是怎么实现的

抽象

Server
 --Service
  --Connector
  --Engine
   --Host
    --Context
    --Context
   --Host
 --Service

acceptor-->worker-->Http11Processor-->

connector-->socket-->HTTPProtocolHandler-->

## Ref

[Which Version Do I Want?](http://tomcat.apache.org/whichversion.html)

[Where Did Tomcat Come From?](https://www.oreilly.com/library/view/tomcat-the-definitive/9780596101060/ch01s05.html)


