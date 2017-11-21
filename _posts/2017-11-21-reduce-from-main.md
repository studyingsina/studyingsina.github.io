---
layout: post
title: "Java多线程系列-开始给主线程减压"
description: "Java 多线程 线程池"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![Reduce-Main](/css/pics/2017-11-21-reduce-from-main.jpg)

# 背景

接着上一篇[《Java多线程系列-先写一个简单WebServer》](http://www.longtask.net/2017/11/20/a-simple-webserver/)我们提出的问题，main线程做了所有工作，为了更好的处理请求，我们给main线程减压，加入一个工作线程分担main线程处理HTTP请求的任务，让main线程只做一些准备性的工作，比如：准备启动参数、环境变量等等.

# 需求

1. 创建一个工作线程去处理HTTP请求，将main线程做的事情和工作线程做的事情分离；
2. 随着功能的完善，用面向对象的思想去组织我们的代码，抽象出一些概念；而不是像V1版本所有的代码都放在一个类中；

# 功能开发

## 加入工作线程

此时，我们开始进入Java Thread编程了。

主要代码如下：

 ```Java

/**
 * 启动处理线程.
 */
private void start(BootstrapV2 server) {
    logicThread = new Thread(new Worker(server));
    logicThread.setName("logic-process-thread");
    logicThread.start();
}

/**
 * 处理HTTP请求的工作者.
 */
public class Worker implements Runnable {

    private BootstrapV2 server;

    public Worker(BootstrapV2 server){
        this.server = server;
    }

    @Override
        public void run() {
            server.serve();
        }

}

```

完整代码实现：[BootstrapV2.java](https://github.com/studyingsina/concurrency-programming-demo/blob/master/src/main/java/com/studying/concurrency/v2/BootstrapV2.java)

## 做个重构
随着功能的不断完善，我们要代码量也越来越多，将所有代码耦合在一个类中显然不是一个好的办法，这一版当中用面向对象的思想去做个重构，关键点：职责单一；

### Bootstrap-启动器

启动器，只做WebServer启动的工作，比如准备参数、环境变量、启动工作线程等，不做具体处理HTTP请求的任务；

完整代码实现：[Refactor](https://github.com/studyingsina/concurrency-programming-demo/tree/master/src/main/java/com/studying/concurrency/v2/refactor)

```Java

/**
 * Created by junweizhang on 17/11/21.
 * 第二版 WebServer 重构,将main线程和服务线程分离.
 * 抽象出三个角色:
 *      Bootstrap-启动器
 *      WebServer-Web服务器
 *      Worker-处理HTTP请求的工作者.
 */
public class Bootstrap {

    /**
     * 我是启动器,只做参数初始化等相关工作.
     * @param args
     */
    public static void main(String[] args) {
        try {
            int port = 8080;
            String docRootStr = "htmldir";
            URL url = Bootstrap.class.getClassLoader().getResource(docRootStr);
            File docRoot = new File(url.toURI());
            WebServer webServer = new WebServer(port, docRoot);
            Logs.SERVER.info("init webServer : {}", webServer);
            Logs.SERVER.info("我是main线程, 好开心, 我已经被释放出来了, 可以做些其它的事情...");
        } catch (Exception e) {
            Logs.SERVER.error("main start error", e);
            System.exit(1);
        }
    }


}

```

### WebServer-Web服务器

WebServer，负责WebServer对外提供HTTP的服务，比如持有ServerSocket、将具体任务的执行代理给工作线程等；

```Java

public class WebServer {

    private ServerSocket ss;

    private File docRoot;

    private boolean isStop = false;

    // 处理HTTP请求线程
    private Thread logicThread;

    public WebServer(int port, File docRoot) throws Exception {
        // 1. 服务端启动8080端口，并一直监听；
        this.ss = new ServerSocket(port, 10);
        this.docRoot = docRoot;
        start(this);
    }

    /**
     * 启动处理线程.
     */
    private void start(WebServer server) {
        logicThread = new Thread(new Worker(server));
        logicThread.setName("logic-process-thread");
        logicThread.start();
    }
    ...
}

```

### Worker-处理HTTP请求的工作者

Worker，工作者，监听8080端口，处理HTTP请求，这里我将其作为WebServer的内部类；

```Java

/**
 * 处理HTTP请求的工作者.
 */
public class Worker implements Runnable {

    private WebServer server;

    public Worker(WebServer server){
        this.server = server;
    }

    @Override
        public void run() {
            server.serve();
        }

}

```

# 问题

到这一版，我们已经实现main线程和工作线程的分离，使其各司其职，但是还存在着问题：

1. 工作线程还是既监听8080端口，又处理HTTP请求；
2. 因为工作线程做的事情太多，所以效率不高，我们请求依旧是串行化处理；

# 感想

经过这一版，我们的代码相对来说清晰一些了，并且也开始接触Java Thread编程了，虽然只是简单的new Thread()，慢慢来；

