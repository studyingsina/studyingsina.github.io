---
layout: post
title: "设计模式-责任链之抛砖引玉"
categories: Tech
tags: Tech
---

* content
{:toc}

![石油管道](/css/pics/2017-06-11-chains.jpg)

## 背景

最近遇到几处使用了责任链模式的地方，所以想总结一下。

## 讲个故事
上小学的时候，班里经常会有同学说，帮我给个纸条给那谁谁谁，比如坐第一排的白居易同学新写了一首诗，要传纸条给坐在第五排的刘禹锡同学炫耀一下，于是通过中间这一个个同学传递，便是一个典型的责任链模式;

另一个故事，在[《who build america》](https://movie.douban.com/subject/20277091/)纪录片中，石油大王Rockefeller为打破运输渠道被铁路大亨Vanderbilt垄断的局面，出资修建布满全国的石油管道，这一节节的石油管道，也是一个典型的责任链模式；

由此可见，责任链模式的思想在生活中处处可见；

## 是什么

责任链模式到底是什么？

* 不就是烤串么？
* 不就是水管么？
* 不就是链表么？
* ...

对，上边的这些比喻都没问题，问题的关键是学以致用；往往当我们在实际的业务中，遇到类似场景却不知道如何应用了；

## 有什么用

不用责任链模式有啥问题呢？用if else、for each信手拈来、照样实现业务逻辑、照样上线运行代码、照样拿工资，没准比那些使用责任链模式的人拿的薪资还高；

不错，如果将上边这句话中的`责任链模式`换成其它任意一种设计模式，都是说的通的，并且在一些使用函数式编辑语言的同学眼中、设计模式这东西根本就不应该存在，那我们还有必要用责任链模式么？

在我看来，使用责任链模式有一些好处：提升代码扩展性、更容易做到单一职责；当然，前提是得用对场景；

## 举例

对于广大程序猿来说，说一万遍理论不如直接上实例，下面我们就以几个经典开源项目为例，介绍下责任链这个模式，用最后附上一个我在实际业务开发中用到的场景；

### Unix

当我在自己home目录下执行`ls | grep c | sort -r` 这个命令组合，即先通过`ls`命令查找home目录下的所有文件及文件夹，再通过`grep`命令查找到包含字母c的文件、文件夹名，再通过`sort`命令根据文件、文件夹名倒序排列；

```
> ls | grep c | sort -r
workspace
Public
Pictures
Music
Documents
CLionProjects
Applications
AndroidStudioProjects
```

在这个命令组合中，符号`|`起到了一个串连的作用，通过标准的接口将其中的每个命令（`ls` `grep` `sort`）串连起来，而每个命令就是责任链中的一个节点，每个节点做好自己的一件事情，通过标准的接口将输出交给下一个节点直至结束；

做一件事、把它做好；（这是不是就是单一职责原则）

### Servlet

在Java Servlet规范中，有这么两个接口：`FilterChain` 和 `Filter`，比如下面这个web.xml配置了两个`Filter`，

```
<!-- HTTP请求编码处理的Filter  -->
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>
        org.springframework.web.filter.CharacterEncodingFilter
    </filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

<!-- HTTP跨域请求处理的Filter  -->
<filter>
    <filter-name>crosFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>crosFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

用图来描述一下上边的配置：

![chains-servlet](/css/pics/2017-06-11-chains-servlet.jpg)

当HTTP请求到达时，Servlet容器（比如大家常见的Tomcat、Jetty等）都会将`web.xml`中配置的节点（即`filter`）串连起来，一个个执行，其中每个节点执行完自己的任务将请求交由下一个节点执行；

Servlet容器实现了`FilterChain`，我们只需要实现自定义`Filter`，来看看这两个接口声明：

```
// 要实现自己的逻辑。
public interface Filter {

    /**
    * 初始化方法
    */
    public void init(FilterConfig filterConfig) throws ServletException;

    /**
    * 执行具体的逻辑，并声明各节点统一的方法
    */
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;

    /**
    * 销毁方法
    */
    public void destroy();

```

```
// 容器去实现责任链，将每个节点串连起来
public interface FilterChain {
	
	/**
	* Causes the next filter in the chain to be invoked, or if the calling filter is the last filter
	* in the chain, causes the resource at the end of the chain to be invoked.
	*
	* @param request the request to pass along the chain.
	* @param response the response to pass along the chain.
	*/
    public void doFilter(ServletRequest request,ServletResponse response) throws IOException, ServletException;

}

```

Servlet规范给我们提供了一个比较标准的接口声明，为什么这个地方可以用责任链模式？来分析下使用场景：一次HTTP的执行过程，是一个典型的请求/应答模式，在执行过程中，不同的业务可能会有不同的逻辑，比如应用A需要有跨域的处理，应用B需要有黑白名单鉴权的处理，在整个HTTP执行流程中我们需要能灵活的配置不同的服务节点，用责任链很好的解决了面临的需求；

此处，`FilterChain`就类似unix命令中的`|`符号的作用，而`Filter`就类似`ls`、`grep`这样的具体命令，每个`Filter`都一个统一的方法`doFilter`。

### Tomcat

Tomcat是Servlet规范的一个实现，被人称为Servlet容器，当然Tomcat也是一个Web容器，能处理Web请求，作为Web容器，Tomcat要将浏览器传过来的HTTP网络流进行解析处理。

场景是这样：当一个HTTP请求到达时，在Web处理这一层，Tomcat要做很多的工作，比如记录访问日志、请求加解密、SSL认证等工作，并且这些工作有些业务是需要、有些业务是不需要的，需要Web容器能灵活的配置，于是Tomcat再一次的使用了责任链模式。

Tomcat抽象出来了一个`Valve`这个接口：声明责任链中的节点，抽象出来了一个`Pipeline`：将责任链中的所有节点串连起来，就像这两个单词的字面意思一样，`Valve`：水龙头、阀门，`Pipeline`：水管、管道，看图：

![tomcat-vavle](/css/pics/2017-06-11-tomcat-vavles.jpg)

其实这张图和上边Servlet是一样，我们再来看下Tomcat是如何声明这两个接口的：

```
package org.apache.catalina;

import java.io.IOException;
import javax.servlet.ServletException;
import org.apache.catalina.connector.Request;
import org.apache.catalina.connector.Response;

/**
* 每一个Vavle都是责任链上的一个节点，承担具体的功能。
*/
public interface Valve {

    public String getInfo();
    
    public Valve getNext();

    public void setNext(Valve valve);

    /**
    * 责任链中每个节点统一的接口
    */ 
    public void invoke(Request request, Response response) throws IOException, ServletException;

    public void event(Request request, Response response, CometEvent event) throws IOException, ServletException;

}

```

每个节点都要实现这个接口，具体的功能在`invoke`方法中实现；

Tomcat内部有几十个这种`Vavle`的实现，其实Tomcat处理HTTP的核心流程都是用一连串`Vavle`实现的，如图：

![tomcat-vavle](/css/pics/2017-06-11-tomcat-vavles2.jpg)

再来看`Pipeline`：

```
package org.apache.catalina;

/**
* 管道，将责任链中的所有节点串连起来，决定了节点之间的调用顺序。
*/
public interface Pipeline {

    public Valve getBasic();

    public void setBasic(Valve valve);

    public void addValve(Valve valve);

    public Valve[] getValves();

    public void removeValve(Valve valve);

    public Valve getFirst();

}

```

这里Tomcat在管道中又加入了自己的一些操作，这里我们不做过多介绍，`Pipeline`的核心作用是将节点串连起来，其实内部是通过类似单向链表来实现的，当然你也可以其它数据结构来实现；

此案例中，`Pipeline`就类似unix命令中的`|`符号的作用，而`Valve`就类似`ls`、`grep`这样的具体命令，每个`Valve`都有一个统一的调用方法`invoke`。

### Netty

Netty是Java领域的一个开源IO通信组件，不少开源项目都用Netty作底层通信，比如阿里的RPC框架Dubbo，Netty有一个IO事件处理流；

场景是这样的：每一次IO交互，都是一个IO读、写操作流（在Java中被称为`Channel`），而Netty会将读、写封装成I/O事件进行处理，比如编码、解码、报文压缩等，Netty是如何应用责任链模式的呢？它抽象出`ChannelPipeline`、`ChannelHandler`这两个接口，`ChannelHandler`是责任链上处理具体逻辑的节点，`ChannelPipeline`将这些节点串连起来，看Netty自己的源码注释便一目了然：

```
 * <pre>
 *                                                 I/O Request
 *                                            via {@link Channel} or
 *                                        {@link ChannelHandlerContext}
 *                                                      |
 *  +---------------------------------------------------+---------------+
 *  |                           ChannelPipeline         |               |
 *  |                                                  \|/              |
 *  |    +---------------------+            +-----------+----------+    |
 *  |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  |               |
 *  |               |                                  \|/              |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  .               |
 *  |               .                                   .               |
 *  | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
 *  |        [ method call]                       [method call]         |
 *  |               .                                   .               |
 *  |               .                                  \|/              |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  |               |
 *  |               |                                  \|/              |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  |               |
 *  +---------------+-----------------------------------+---------------+
 *                  |                                  \|/
 *  +---------------+-----------------------------------+---------------+
 *  |               |                                   |               |
 *  |       [ Socket.read() ]                    [ Socket.write() ]     |
 *  |                                                                   |
 *  |  Netty Internal I/O Threads (Transport Implementation)            |
 *  +-------------------------------------------------------------------+
 * </pre>

```

其实Netty做了更丰富的抽象，比如抽象出`Inbound Handler`、`Outbound Handler`，这里我们不做过多介绍；

### 实际业务系统中应用

实现设计模式，比猫画虎我们都会，关键是用对场景、学习致用，这里我举一个在业务开发中用的地方，不一定合适、抛砖引玉、仅供参考；

场景：有一个业务量比较小的订单系统，日订单量几千到几万的样子，在下单的操作时，有如下业务逻辑：参数转换、参数校验、逻辑校验、风控、锁库存、创建订单、触发订单事件等操作，并且之后随着业务需求的变化还可能加入其它逻辑如幂等判断、限流、防抓取，整个过程如下图：

![order-create](/css/pics/2017-06-11-order-create.jpg)

这里我抽象出`PipelineService`、`VavleService`两个接口，其实从名字就可以看出，是从Tomcat那里学来的，`VavleService`是责任链中执行业务逻辑的节点，`PipelineService`将这些节点串连起来；来看这两个接口的声明：

```
/**
 * 1. vavle 可以排序.
 * 2. 可以跳过其中某些vavle不执行.
 * 3. 可以在某个vavle上中断执行流程.
 * 4. 需要有一个返回结果,提供给上游使用.
 * 5. 不同场景可以定义不同的执行链.
 * 6. 前一个vavle的结果,可以给后边的vavle使用.
 * 7. 支持vavle的回滚操作.
 */
public interface VavleService {

    VavleResult execute(PipelineRequest request, PipelineResponse response) throws Exception;

    String getName();

}

/**
 * 将责任链中每个节点串连起来。具体实现上就是一个ArrayList，并且加上了跳表的功能。
 */
public interface PipelineService {

    void start(PipelineRequest request, PipelineResponse response);

}
```

在业务中下单的逻辑都是通过`VavleService`的子类来实现的，如下：

```
<!-- XX业务下单流 -->
<bean id="userCreatePipelineService" class="com.service.pipeline.UserPipelineService"
      scope="prototype">
    <property name="vavleList">
        <list>
            <ref bean="orderBaseParamCheckVavleService"/>              <!-- 校验订单基本参数 -->
            <ref bean="orderRepeatCheckVavleService"/>                 <!-- 校验重复下单 -->
            <ref bean="createParamConverterVavleService"/>             <!-- 填充订单参数 -->
            <ref bean="orderParamCheckV2VavleService"/>                <!-- 校验子下单参数 -->
            <ref bean="logicCheckVavleService"/>                       <!-- 订单金额校验 -->
            <ref bean="riskControlVavleService"/>                      <!-- 风控校验 -->
            <ref bean="generateOrderSnVavleService"/>                  <!-- 生成订单唯一编号 -->
            <ref bean="stockLockService"/>                             <!-- 锁库存 -->
            <ref bean="orderCreateVavleService"/>                      <!-- 生成订单 -->
            <ref bean="orderStatusChangeEventVavleService"/>           <!-- 订单状态变更事件 -->
            <ref bean="createToPayAdapterVavleService"/>               <!-- 下单到支付参数转换 -->
            <ref bean="payCheckVavleService"/>                         <!-- 支付参数校验 -->
            <ref bean="payApplyVavleService"/>                         <!-- 请求预支付 -->
            <ref bean="orderStatusChangeOldVavleService"/>             <!-- 订单状态变更事件 -->
        </list>
    </property>
</bean>
```

如果之后在业务需求要再加其它逻辑，去写一个`VavleService`的实现类即可，并且节点之间的顺序可灵活调整，在新的版本中我们还加入了支持某节点业务回滚的操作如：锁库存成功但是生成订单失败、此时要返还库存；

此处下单场景使用责任链，不一定合适，但多了一种尝试，写`if else`也是写，换种思路也是写，当然，前提是保障业务正常；

## 总结

很多优秀的开源框架都会用到设计模式，正所谓：无模式不框架，学习设计模式，一个比较好的途径就是看别人如何使用。

但是设计模式真的有用么？相信一百人心中会有一百个答案，觉得有用就了解下、觉得没有也无所谓，依旧不影响日常工作，当`if else`嵌套的实在难以忍受再来抽象一层也未尝不可。

