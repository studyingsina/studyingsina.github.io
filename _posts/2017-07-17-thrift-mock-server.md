---
layout: post
title: "Thrift Mock Server"
description: "Thrift Mock Server 思路 搭建"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![Mock](/css/pics/2017-07-17-jewellery.jpg)

## 问题
当我们在写自动化case的时候，往往需要将外部的依赖mock掉，在我们公司，RPC组件用的Thrift，那么就涉及到一个问题，如何将Thrift给mock掉，并且我们这里是在做自动化测试（功能测试），不是单元测试，在单元测试的时候我们很容易mock。

自动化测试流程如下：

![autotest-process](/css/pics/2017-07-17-autotest-process.png)
1. 准备要Mock的数据传给Mock Server缓存起来；
2. 测试对外提供的服务接口(比如订单系统提供给C端用户的下单接口)；
3. 订单服务会调用外部依赖（此处是Mock Server的Thrift依赖，如查询产品接口、支付接口）；
4. Mock Server会将第1步Mock的数据返回给业务系统；

由此我们便达到测试自己系统的功能而不受外部依赖环境的影响，不用再关心外部接口是否可以、返回数据是否一致，因为我们将所有的外部依赖都Mock掉。

## 难点
Thrift自定义了一套协议，根据IDL生成的Thrift类在服务调用端、服务提供端都需要，并且Thrift协议是按生成的类来封装字节，一个端口对应一个服务，如何在仅提供一个端口的情况下Mock多个服务？

## 思路

### Thrift调用流程
我们先来了解下Thrift调用的流程，如图：

![thrift-invoke](/css/pics/2017-07-17-thrift-invoke.png)

从图中可以看到，我们在很多点都可以做一些动作，达到我们Mock的效果；凡是图上标记数字(1~14)的调用处都可以；

整体上可以分为两类：Client端、Server端。

### Client
在Thrift Client端做一层代理，将Client的请求都打到一个Mock Server端口上，在代理层中解析Mock的数据；如图：

![thrift-mock-client](/css/pics/2017-07-17-thrift-mock-client.png)

* 优点：容易实现；
* 缺点：业务系统要引入Thrift Client的代理，有侵入；
* 总结：重Client、轻Server；

### Server
在Thrift Server端缓存Mock的数据，伪造成Thrift协议返回；

* 优点：业务系统无侵入；
* 缺点：实现难度要大一些；
* 总结：轻Client、重Server；

## 实现
理论上以上两种思路都可行，但是作为一个业务系统开发人员来说，我个人更倾向于Server端这种，因为这种方式对业务系统无侵入，更接入于真实的测试环境，尽管这种方式我们要比Client多做一些工作；

### Client

对Thrift生成的Iface做一层代理，
业务方如下使用：

```Java
    // 生成Iface的Proxy.
    HelloWorldService proxyService = HelloWorldServiceProxy();
    
    // 调用say方法.
    Result ret = proxyService.say();
```

HelloWorldServiceProxy的say方法如下实现：

```Java
    // 代理类的实现
    public Result say() throws TException {
        // 获取Mock数据,比如通过http接口获取Mock Server的Mock数据.
        mockData = httpClient.getMockData();    
        // 将Mock数据解析为Result对象
        Result ret = paseMockDataToResult();
        // 返回结果
        return ret;
    }
```
当然了，这里只是简单介绍一种思路，具体实现上见仁见智。

### Server
说下Server端的实现，Thrift其实底层也还是对Java Socket的包装，只是在原生的Socket上面做了一些约定（即Thrift Protocal）：比如前几个字节是TMessage头，后几个字节是TMessage尾等等，如果想清楚这点，那么我们实现起来就简单了。

按照我们上图Thrift调用流程中，从8到11这4步我们都可以做些动作，此处我选择在步骤10处返回Mock数据、直接操作Socket的InputSream和OutputStream，这样我可以不关心Thrift协议层的一些细节而达到目的；

![thrift-mock-server-me](/css/pics/2017-07-17-thrift-mock-server-me.png)

贴一段主要的代码逻辑：
```Java

    private void mockDaynamicGetStrResult(TProtocol oprot, TMessage msg) throws TException {
       try {
           // 1. Mock返回数据,实际情况下Mock数据可以由Test Case通过HTTP接口传过来,Mock Server缓存.
           TestThriftService.getStr_result getStr_result = new TestThriftService.getStr_result();
           ResultStr resultStr = new ResultStr();
           resultStr.setValue("mockDaynamicGetStrResult......");
           getStr_result.setSuccess(resultStr);
    
           TFramedTransport tFramedTransport = (TFramedTransport) new TFramedTransport.Factory().getTransport(null);
           TBinaryProtocol tBinaryProtocol = (TBinaryProtocol) new TBinaryProtocol.Factory().getProtocol(tFramedTransport);
           tBinaryProtocol.writeMessageBegin(new TMessage(msg.name, TMessageType.REPLY, msg.seqid));
           getStr_result.write(tBinaryProtocol);
    
           // 2. 获取到Transport中的 writeBuffer_
           Field writeBuffer = ReflectionUtils.findField(TFramedTransport.class, "writeBuffer_");
           logger.info("writeBuffer : {}, accessable : {}", writeBuffer, writeBuffer.isAccessible());
           ReflectionUtils.makeAccessible(writeBuffer);
           logger.info("accessable : {}", writeBuffer.isAccessible());
           TByteArrayOutputStream fakeOutputStream = (TByteArrayOutputStream) writeBuffer.get(tFramedTransport);
           byte[] bytes = fakeOutputStream.get();
    
           // 3. 偷梁换柱,将Mock Byte[] set到OutputStream中.
           TByteArrayOutputStream outputStream = (TByteArrayOutputStream) writeBuffer.get(oprot.getTransport());
           logger.info("outputStream : {}", outputStream);
           outputStream.reset();
           outputStream.write(bytes);
       } catch (Exception e) {
           logger.error("mockDaynamicGetStrResult error", e);
       }
    }

```

此处我实现了一个Server端的[demo](https://github.com/studyingsina/spring_use):

先运行ThriftServerDemo.java，再运行ThriftClientDemo.java，便可看到结果；

此处只是拿原生的Thrift写了一个实现，离实际正式环境使用还有一定差距，不过思路是一致的，比如我们公司对原生Thrift做了一层封装，加入了服务自动注册、调用链追踪等功能，我便按上边的思路实现了一个Mock Server。

### 其它
对于其它一些Mock Server实现方式，我见过的有：

* 上传Thrift IDL自动生成Thrift类、配置端口再通过Shell等脚本启动一个Mock服务；
* 将业务系统依赖的所有外部Thrift接口Jar都引入进来，所有接口都实现一遍，这种方式Mock Server依赖的Jar会越来越多，每个服务都需要配置一个端口，对于一个小组还可行，但是对于整个公司级别的Mock Server便不可行了；

## 总结
以上为自己做业务系统时遇到的一个Mock问题及想法，不管哪种实现，没有一种标准答案，解决问题达到效果即可，心即是理。
