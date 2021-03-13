---
layout: post
title: "系统压测"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![stree-testing](/css/pics/2019-08-01-stress-testing.jpeg)

## 背景

业务系统缺少压测

### 为什么要压测

两个字：预防

一句话：预防胜于治疗

![stree-testing-why](/css/pics/2019-08-01-stress-testing-why.jpeg)

为应对流量高峰，通过压测提前发现系统短板，提前预防故障；

其实这是个资源成本问题；做个假设：如果预防要花费10w、而出了故障也就损失1w，那么没必要做预防、没必要压测；

### 压测能解决什么

只能能解决部分问题，常见的比如：机器资源不足提前扩容、线程池不足提前扩容等；扩容类的问题；

减少故障而不能避免故障；

### 压测不能解决什么

不能解决所有问题，常见的比如：外部依赖挂掉该出故障还是要出故障；

## 如何压测

![stree-testing-how](/css/pics/2019-08-01-stress-testing-how.png)

其实这么拆分是以组织结构为维度来说的，一般以一个组织结构/小组为单位，比如促销业务模块会拆分成多个小应用，如红包应用、返减应用等，而多个业务模块串起来就是整个链路，比如api模块、产品模块、商家模块、订单模块、促销模块、支付模块等串起来实现一个完整的业务链路；

### 单应用

此处的单应用指一个独立部署运行的服务，比如订单查询服务；

对单应用的压测，通常针对某些接口进行，可将外部依赖mock掉，只压系统内部的问题；

### 多应用

此处多应用指多个单应用之间有调用关系，通常一个小组内部负责的服务，比如订单模块（假如服务是这么拆分的，这个跟组织结构和拆分粒度有关）：

![stree-testing-multi](/css/pics/2019-08-01-stress-testing-multi.png)

### 全链路

整个链路，比如Api--->下单--->支付--->退款整个业务链路，通常需要多个团队一起配置；

## 遇到的问题

一般读压测相对简单一些，准备好数据、将需要的依赖mock掉就可以开始，对于写压测会麻烦些，主要是会造成写压测数据、压测后还要清理压测数据，对于一些外部依赖还要做写mock，如支付接口、回调接口；说下我在做主流程写压测过程中遇到的一些问题及解决方法；

### 写压测

压测流程一般为：

0. 压测环境准备，系统可能还需要改造下、机器隔离等
1. 准备压测数据
2. 配置压测参数，如压测时间、是否阶梯压测等
3. 系统指标监控

业务场景大概如下：上游RPC推数据到服务A--->服务A发MQ--->服务B监听MQ进行处理（内部有N多外部依赖及写数据操作）--->服务B再调用服务C外部请求（C内部也有外部依赖及写数据操作），如下图：

![stree-testing-us](/css/pics/2019-08-01-stress-testing-us.png)

做链路压测的一个核心关键点是在整个链路中要标识出压测流量，这样我们才能针对压测数据做特殊处理，如何标识出压测流量，需要我们对压测数据做标识，原理类似分布式追踪；这需要公司中间件的支持，比如MQ、Cache、数据库中间件、压测平台、RPC组件等等；

### 写DB

创建影子表，当压测流量来的时候数据全部写入影子表，将压测数据和真实数据分离；

### 写Cache

创建影子Cache，当压测流量来的时候数据全部写入影子Cache；

### MQ

MQ支持开关配置，开关控制是否处理MQ，如果处理则按正常MQ转发，如不处理则忽略掉，开关支持Topic和消费组粒度；

### 外部依赖

现在微服务横行，一个应用没几个外部依赖都不好意思出门跟人家打招呼，对于外部依赖，如有需要则要Mock掉，比如我们录制了线上流量开始压测，但是压测的时候我们要查询订单状态（外部依赖），此时订单状态已经变更，而我们业务逻辑需要满足一定状态的订单才处理，为了让流程正常走下去，我们需要将订单状态Mock掉，此时有多种Mock方式，Mock外部依赖或本地方法皆可；

### 压测标识

压测整个链路的关键在压测标识，所以压测标识不能丢，整个链路上除了一些中间件要支持压测标识透传，我们应用中也不能丢，特别是对于应用内部有异步线程处理的时候，父线程需要将压测标识传递下去；

