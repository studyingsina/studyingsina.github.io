---
layout: post
title: "一次HTTP请求在Tomcat中的执行流程"
categories: 
- Life
tags:
- Life
---

* content
{:toc}

![A-HTTP-Request-In-The-Tomcat](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.001.jpeg)

## 背景

组内要求每人都要进行分享，故准备一个组内的小分享，因为实际工作中使用过Tomcat和Jetty作为Servlet容器，所以先来熟悉下Tomcat，当然现在使用Jetty更多；

通过了解一次HTTP请求在Tomcat中的流转来认识Tomcat内的重要组件及设计；

![tomcat002](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.002.jpeg)

## 需求

假设时间回到20年前，你是[Sun Microsystems](https://en.wikipedia.org/wiki/Sun_Microsystems)的一名开发人员，产品经理[James Duncan Davidson](https://en.wikipedia.org/wiki/James_Duncan_Davidson)给你提了一个需求：

### 需求背景

为了推广Sun的Servlet规范，让广大程序员喜欢用Servlet开发应用、降低程序员的开发成本，需要提供一个开源的Servlet容器；

### 需求内容

1. 支持Servlet+JSP规范
2. 支持WebServer
3. 开源
4. 易扩展
5. 高性能
6. 自定义/被替换

## 实现方案

### Demo版实现方案

先写一个Demo，看实现一个WebServer+Servlet容器要做哪些最基本的事情，如下图：

![tomcat004](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.004.jpeg)

简化下Demo版的逻辑：

1. 启动服务器监听端口,如8080
2. 开始监听客户端请求,如http请求
3. 解析http请求参数
4. 业务自己逻辑处理,如增删改查
5. 将返回数据按http协议格式返回

在Tomcat中也存在这5步，只不过作为一个可广泛应用的Server，Tomcat考虑的逻辑更多更周全。

### 功能拆分

先把变和不变的逻辑拆分开来，哪些是易变的逻辑、哪些是不易变的逻辑？

变：第[1、2、3、5]步骤是不易变的，基本上每一个服务器都要做的事情；

不变：第[4]步是每个业务都不一样的，如下图：

![tomcat011](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.011.jpeg)

先把第[4]步放一放、因为这是应用层的事情，对[1、2、3、5]步骤再进行拆分：

+ 第1步启动服务器监听端口较为简单，new一个ServerSocket，
+ 第3和5步骤是对HTTP请求/输出流的解析和封装、属于一类事情，我们将3、5放在一起，如下图：

![tomcat012](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.012.jpeg)

### 性能

从线程的角度来看，目前只有一个应用启动线程（main线程）在处理监听端口-解析http请求的事情，这样当请求多的话、main线程的处理效率成为瓶颈，所以我们将监听端口和请求处理分开，如下图：

![tomcat013](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.013.jpeg)

一共三类线程，main线程、监听线程、工作线程池，处理模型是这样的：

1. main线程启动ServerSocket，并创建一个监听线程（Acceptor）+一个工作程线池（多个Worker）；
2. 监听线程只用来监听Socket，收到Socket后放入线程池的队列中（Queue）；
3. 每一个工作线程（Worker）处理各自的HTTP请求（Socket），Worker将解析/封装HTTP协议的事交由Handler处理；

### Servlet

上图的Controller其实就对应Servlet，如下图：

![tomcat014](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.014.jpeg)

以HTTP请求流转的视角将上图简化一下，如下图：

![tomcat015](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.015.jpeg)

### 四层抽象

+ 一个Servlet（Tomcat内部叫作Wrapper）可以处理一个或一类HTTP请求，有时候我们需要在一个容器内部署多个Servlet，所以容器要支持多个Servlet的情况；
+ 多个Servlet如果认为是一个应用（Tomcat内部叫作Context）的话，有些时候我们希望一个服务器能够部署不止一个应用；
+ 多个应用如果认为是一个应用集（Tomcat内部叫Host）的话，那么有些时候我们希望一个服务器能够部署不止一个应用集；
+ 多个应用集如果认为是一个服务引擎（Tomcat内部叫Engine）的话，那么我们先打住；

Tomcat对于Servlet支持层级的抽象、分为了4层：

![tomcat016](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.016.jpeg)

+ 一个Engine下可以有多个Host，类比为一个服务器中可以部署多个域名，如公司外部域名、公司内部域名
+ 一个Host下可以有多个Context（应用），类比为一个域名下可以部署多个服务，如用户服务、订单服务、商品服务
+ 一个Context下可以有多个Wrapper（Servlet），类比为一个应用下可以有多个请求链接，如/user/info、/user/friends
+ 一个Wrapper对应一个Servlet，一个Servlet对应一个请求链接，如UserInfoServlet对应的请求链接是/user/info

将上图换一种方式描述，如下：

![tomcat017](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.017.jpeg)

当Engine从上游收到请求，Engine先判断要交给哪个Host--->Host再判断要交给哪个Context--->Context再判断要交给哪个Servlet处理；

打个比方，XX组织要求公司的支持IPV6标准，找到公司CEO：

+ CEO判断这个事该CTO负责（而不是CFO、或其它的CXO）；
+ CTO接收到这个需求判断应该由基础网络工程部处理（而不是其它技术部门）；
+ 层层分解、并且每层将任务派发下去时会附近一些指示等等；

既然每一层（Engine-->Host-->Context-->ServletWrapper）将请求往下分派时都会对请求做一部分加工处理，那么Tomcat是如何处理这种场景呢？没错，就是责任链模式，如图：

![tomcat018](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.018.jpeg)

Pipeline是责任链、责任链中可以配置多个过滤器-Vavle，不过Tomcat的责任链做了一点小改造，每个责任链中至少要配置一个默认的过滤器，这个过滤器承担的职责是将请求传递到下一层级的Pipeline中，请求流是这样流转的：

+ 上游的请求request-->Pipeline(Engine的责任链)-->自定义各种过滤器（Engine的）-->EngineVavle（Engine的默认过滤器）-->
+ Pipeline（Host的责任链）-->自定义各种过滤器（Host的）-->HostVavle（Host的默认过滤器）-->
+ Pipeline（Context的责任链）-->自定义各种过滤器（Context的）-->ContextVavle（Context的默认过滤器）-->
+ Pipeline（ServletWrapper的责任链）-->自定义各种过滤器（Wrapper的）-->WrapperVavle（Wrapper的默认过滤器）-->Servlet

这4层每一层都有一个责任链，然后4个责任链串到一起来将请求传递到Servlet，如图：

![tomcat019](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.019.jpeg)

当最后的WrapperVavle拿到请求后，要把请求传递给Servlet，但是我们都知道，Servlet规范是支持在请求到达Servlet之前也配置过滤器（Filter）的，其实是就Sun在制定Servlet规范时，在应用层给了广大程序员提供可扩展手段，要求Servlet容器再实现一个应用层的责任链（过滤器链），公司内部的一些SSO、权限校验等都会用到Filter实现，如下图：

![tomcat020](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.020.jpeg)

### 整合一下

我们将前边提到的这些概念整合一下、如图：

![tomcat021](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.021.jpeg)

上图基本描述了整个请求从进入Tomcat到返回结果流经的路线；

但是这样还不够，我们需要将上图这些组件进行组装、抽像，将左侧方框中Acceptor、Queue、ThreadPool、Handler这些组装起来，叫做Connector（连接器：连接浏览器来的请求）；

然后右侧方框中这些抽像起来叫做Container（容器），Wrapper是小容器、Context是中容器、Host大容器、Engine是特大容器，如图：

![tomcat022](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.022.jpeg)

如果我们再抽像一层，将Connetor和Container组装到一起呢，是不是就可以组成一个服务（Service），如图：

![tomcat023](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.023.jpeg)

如果多个Service组合起来再抽像一层，可以称为一个服务器（Server），如图：

![tomcat024](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.024.jpeg)

把上图和Tomcat的配置文件放一起对比下，上边提到过的概念基本上在配置文件中都有对应的标签：

![tomcat025](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.025.jpeg)

```
Server
 --Service
  --Connector
  --Engine
   --Host
    --Context
    --Context
   --Host
 --Service
 ```
### Debug源码验证

下边我们通过Debug一个HTTP请求，看下请求过程中的组件（上边提到的Sever、Service、Connector、Container等）及调用堆栈，

先来看下Server，如图：

![tomcat026](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.026.jpeg)

上图显示：Server对象中可以有多个Service、Service对象中可以有多个Connector+一个Engine；

再看Engine，如图：

![tomcat027](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.027.jpeg)

上图显示：Engine对象中可以有多个Host，Engine用一个Map来存储多个Host，图中示例仅一个名叫localhost的Host；

再看Host，如图：

![tomcat028](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.028.jpeg)

上图显示：Host对象中可以有多个Context（应用），Host也用一个Map来存储多个Context，图中示例有5个Context，就是Tomcat本身自带几个应用；

再看Context，如图：

![tomcat029](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.029.jpeg)

上图显示：Context对象中可以有多个Wrapper（Servlet），Context也用一个Map来存储多个Wrapper，图中示例有19个Wrapper，就是Tomcat自带example应用下的demo，Map的key就是web.xml中配置的Servlet名称，value是具体Tomcat生成的Wrapper实例；

再来看下请求的调用堆栈，如图：

![tomcat030](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.030.jpeg)

1. 调用栈最底层是一个Thread；
2. SocketProcessor对Socket进行处理；
3. 其实是将请求交给Http11Processor对Socket中网络Stream进行解析；
4. 然后通过一个适配器（Adaptor）将Connector中的请求传递给Container，Container不会亲自处理请求，而是将请求交给每一层容器的过滤器（一堆invoke方法，每一个都是责任链中的一个过滤器）
5. 在Wrapper的过滤器时生成一个Servlet的过滤器链（FilterChain）
6. Filter走完请求最后交到Servlet中；

在堆栈中我们见到一个叫ErrorReportVavle的过滤器，是Tomcat在Host这一层中内置的，它的作用是干啥呢？还记得Tomcat著名的404错误页面么？没错：

### 经典的404界面

![tomcat031](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.031.jpeg)

这个就是当请求的url不存在时，tomcat返回的默认404界面，来看下它内部的主要实现：

![tomcat032](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.032.jpeg)

其实就是用代码拼出一个404 Not Found的HTML界面，当然我们可以自己去实现一个替换掉默认的。

## 总结

让我们再来回顾一遍整个Tomcat处理HTTP请求的过程：

![tomcat033](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.033.jpeg)

1. 当HTTP请求进来时，Server并不会亲自去处理，而是转交给内部的Service；
2. Service也不会亲自去处理，而是交给Connctor中的Acceptor，Acceptor是专门监听请求的；
3. Acceptor接收到请求（Socket）后，将请求（Socket）放到队列（Queue）中，然后自己又继续监听新的请求；
4. Connector从内部线程池（ThreadPool）中找出一个线程去工作，通过调用HTTP处理器（Handler）去解析HTTP请求；
5. Connector将请求从内部传递到Engine中，Engine将请求交给自己的责任链（Pipeline），责任链又找到内部配置的过滤器；
6. 每个过滤器执行自己内部的逻辑，然后将请求往下一层层传递，直至Servlet内部；
7. 到Servlet后其实就到应用层了，就是广大业务开发同学的领域了，可以写一些增删改查返回结果；
8. 返回结果再一层层向上（和刚才请求进来的流程顺序刚好相反）直至返回上游；

我们回过头来想一想，为何Tomcat把自己内部搞的这么复杂，是为了达到什么目标？像Jetty那样简洁一些不好么？

![tomcat033](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.033.jpeg)

作为一个带有Sun官方基因的开源窗口，面对的使用场景是各种各样的，不同的场景需要有不同的诉求，所以说开源项目要考虑的一个重要因素便是扩展可替换，大到扩展核心组件、小到扩展某一个功能点如日志打印，所以进行这么多抽像封装、设计，比如我们可以切换不同的IO实现、可以替换整个Connector；（仅个人观点）

## 后续

后续大家有啥想了解的，我们可以进一步探讨，如：

![tomcat036](/css/pics/tomcat/A-HTTP-Request-In-The-Tomcat.036.jpeg)

## 八卦



## Ref

[Which Version Do I Want?](http://tomcat.apache.org/whichversion.html)

[Where Did Tomcat Come From?](https://www.oreilly.com/library/view/tomcat-the-definitive/9780596101060/ch01s05.html)

[How Tomcat Works](https://book.douban.com/subject/1943128/)

[apache-tomcat-9.0.36-src](https://github.com/apache/tomcat/tree/9.0.x)

