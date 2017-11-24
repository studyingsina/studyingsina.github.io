---
layout: post
title: "Java多线程系列-增加工作线程-线程池"
description: "Java 多线程 线程池"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![Thread-Pool](/css/pics/2017-11-23-thread-pool.jpg)

## 背景

接着上一篇[《Java多线程系列-缓解忙等问题》](http://www.longtask.net/2017/11/21/reduce-worker/)我们提出的问题，工作线程过多，我们要多加一些线程参与处理任务；

## 需求

1. 增加工作线程数量，一个线程处理任务太慢、太苦逼、太寂寞，有活大家一起干；

## 功能开发

### 简单做

简单点做，直接将原来的一个线程变量改一个线程数组，直接上代码：

```Java

// 处理HTTP请求线程组
private Worker[] workers;

/**
 * 必需先启动工作线程,再启动监听线程.
 */
private void start() {
    // 启动工作线程,工作线程,可以作为守护线程
    int poolSize = 2;
    workers = new Worker[poolSize];
    for (int i = 0; i < poolSize; i++) {
        workers[i] = new Worker(i);
    }
    Logs.SERVER.info("start workerPool size : {} ...", poolSize);

    // 启动监听线程,监听线程,不作为守护线程,保证JVM不退出.
    acceptorThread = new Thread(new Acceptor());
    acceptorThread.setName("http-acceptor-" + port + "-thread");
    acceptorThread.start();
    Logs.SERVER.info("start acceptor thread : {} ...", acceptorThread.getName());
}

```

完整代码实现：[WebServer.java](https://github.com/studyingsina/concurrency-programming-demo/blob/master/src/main/java/com/studying/concurrency/v5/WebServer.java)

### 小重构

现在我们的WebServer可以由多个工作线程处理任务了，不过WebServer所担任的责任有些杂了，其实可以将workers和queue组合起来（组合模式）抽象为一个线程池，工作机制便是监听器线程往线程池中放任务，线程池自己去队列里取任务；

此处我再进行一次小重构，看代码：

```Java

/**
 * 一个简单的固定大小线程池,数组实现,任务先放入阻塞队列,工作线程不断去队列中取任务.
 */
public class ThreadPool {

    // 监听到的socket队列
    private SimpleQueue<Socket> socketQueue;

    // 线程组
    private Worker[] workers;

    public ThreadPool(int poolSize) {
        this.socketQueue = new SimpleQueue<>(3);
        workers = new Worker[poolSize];
        for (int i = 0; i < poolSize; i++) {
            workers[i] = new Worker(this, i);
        }
        Logs.SERVER.info("start workerPool size : {} ...", poolSize);
    }

    /**
     * 由监听线程往队列中放入socket,以备工作线程从中取值进行处理.
     */
    private void assign(Socket socket) throws Exception {
        socketQueue.put(socket);
    }

    /**
     * 工作线程从队列中取出socket.
     */
    private Socket await() throws Exception {
        return socketQueue.take();
    }
}

```

完整代码实现：[WebServer.java](https://github.com/studyingsina/concurrency-programming-demo/tree/master/src/main/java/com/studying/concurrency/v5/refactor)

### 知识点

这次，细心的同学已经发现，我们不能再用notify方法了，改成notifyAll方法了，为什么呢？因为同时等待的工作线程是多个；

还有一个细节是，列队take、put方法开始的判断队列是否为空、是否满的逻辑由原来的if变成了while，想想这是为什么？

## 问题

到这一版，有同学抱怨了，你不是说Java还有JUC的实现方案么，OK，下一版我们就用JUC中的工具去实现：

## 感想

我们现在已经有了一个WebServer的雏形了，想做的完善就一步步迭代，比如想支持Servlet规范，那么再将Socket包装成Request、Response对象，实现Servlet规范便可；

这段时间经常听到中年危机，都说编程是吃年轻饭，太长远我也看不到，做一天是一天，躲进小楼成一统、管他春夏与秋冬。
