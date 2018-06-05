---
layout: post
title: "Java多线程系列-开始给工作线程减压"
description: "Java 多线程 线程池"
categories:
- Tech
tags:
- Tech 多线程
---

* content
{:toc}

![Reduce-Worker](/css/pics/2017-11-21-reduce-worker.jpg)

## 背景

接着上一篇[《Java多线程系列-开始给主线程减压》](http://www.longtask.net/2017/11/21/reduce-from-main/)我们提出的问题，工作线程做了所有的事情，不够专注，就好比我开了一家餐厅，我即当前台接待用户、又当厨师烧菜、又当服务员上菜，等一个用户结账走人后再去接待下一个用户，只有我这一个线程去忙活，苦逼啊，这次，我们就雇一个员工替我分单压力，即再创建一个新的线程去工作。

## 需求

1. 新开一个线程，专门用来监听8080端口，监听到有新的请求Socket后，将Socket交给工作线程去处理请求，这个新的线程我们命名为Acceptor-接收器；
2. 工作线程只处理Socket请求，不做监听的事情；

## 功能开发
这一版我们开始接触线程的一些新知识点：比如引入了共享变量、线程通信，当然也会用synchronized关键字，因为有两个线程，就涉及到对共享变量的读写；

### 加入监听线程

Acceptor-专门监听端口接收Socket，将接收到的Socket交给工作线程处理；

```java

/**
 * 必需先启动工作线程,再启动监听线程.
 */
private void start(WebServer server) {
    // 启动工作线程,工作线程,可以作为守护线程
    workerThread = new Thread(new Worker());
    workerThread.setName("worker-process-thread");
    workerThread.setDaemon(true);
    workerThread.start();
    Logs.SERVER.info("start worker thread : {} ...", workerThread.getName());

    // 启动监听线程,监听线程,不作为守护线程,保证JVM不退出.
    acceptorThread = new Thread(new Acceptor());
    acceptorThread.setName("http-acceptor-" + port + "-thread");
    acceptorThread.start();
    Logs.SERVER.info("start acceptor thread : {} ...", acceptorThread.getName());
}

/**
 * 接收器,监听HTTP端口,接收Socket.
 */
public class Acceptor implements Runnable {

    @Override
    public void run() {
        try {
            while (!isStop) {
                Logs.SERVER.info("acceptor begin listen socket ...");
                Socket socket = listen();
                Logs.SERVER.info("acceptor a new socket : {}", socket);
                assign(socket);
            }
            } catch (Exception e) {
                Logs.SERVER.error("Acceptor process error", e);
        }
    }
}

```
如果你所在网络能访问Gist，那么下边这块代码看起来应该更友好：

{% gist eda3e3f03521f93ed9075bb8229619ad %}

完整代码实现：[WebServer.java](https://github.com/studyingsina/concurrency-programming-demo/blob/master/src/main/java/com/studying/concurrency/v3/WebServer.java)

### 工作线程

工作线程不再监听端口，只去一个固定的地方取Acceptor收到的Socket，这个固定的地方，我们暂时用一个成员变量存储；

```java

/**
 * 处理HTTP请求的工作者.
 */
public class Worker implements Runnable {

    @Override
    public void run() {
        try {
            while (!isStop) {
                Socket s = await();
                if (s != null) {
                    Logs.SERVER.info("worker begin process socket : {}", socket);
                    process(s);
                    socket = null;
                }
            }
        } catch (Exception e) {
            Logs.SERVER.error("Worker process error", e);
        }
    }

}

```

### 知识点

#### 守护线程
工作线程，将其置为守护线程，让它在后台慢慢运行就可以了；

监听器线程，我们将其置为非守护线程，那守护线程和非守护线程有什么区别吗？我们来看下Java Thread中setDaemon方法的说明：

> Marks this thread as either a {@linkplain #isDaemon daemon} thread or a user thread. The Java Virtual Machine exits when the only threads running are all daemon threads.
> This method must be invoked before the thread is started.

如果JVM中所有的线程都是守护线程了，那么JVM就会退出，所以为了不让JVM退出，至少需要有一个非守护线程，这里便是监听器线程；

#### 线程互斥

我们的共享变量会由两个线程去操作，监听器线程去写、工作线程去读，如果在写到一半的时候CPU切换到工作线程去读，那么可能读到的数据为空，所以要保证此处对共享变量的读写都是原子操作，这便是synchronized关键字的作用；

#### 线程通信
此版本涉及两个线程：监听器线程和工作线程，并且这个两个线程是有依赖关系的，即监听器线程先要收到一个Socket，然后再将这个Socket给工作线程，工作线程去处理，这就引出了不同线程之间的通信问题，此处通过一个共享变量来实现，即监听器线程拿到Socket后、交给WebServer.socket变量，然后工作线程从WebServer.socket变量上取值，从这里，你可能已经看到相似的场景了，一个线程接收数据(Socket)、一个线程获取数据(Socket)、有一个地方(共享变量)存储数据，这不是典型的生产者、消费者模型么？别急，这个后边会讲到，此处还是按简单的方式来处理；

线程通信，Java提供的一种方案是wait/nofity机制，我们此处正是用的这种方案：

1. 监听器线程取到Socket后，往共享变量赋值，如果此时共享变量有之前已经赋过的值还没被工作线程取走，那么监听线程就先等待(wait)，如果此时之前赋值的Socket被工作线程取走后、工作线程通知监听器线程可以去赋值了；
2. 工作线程去取Socket时，如果此时共享变量还没有被监听器线程去赋值，那么工作线程就先等待(wait)，如果此时共享变量已经被监听器线程赋值，那么工作线程直接取值即可、并会通知在等待的监听器线程可以再次进行赋值(如果此时监听器线程接收到新的Socket)；

一图胜千言，来看下这个过程：

![Acceptor-Worker-Communication](/css/pics/2017-11-22-worker-acceptor-communication.jpg)

再来看下我们的代码，注释的已经很清楚了：

```java

/**
 * 由监听线程给socket赋值,以备工作线程从中取值进行处理.
 */
private synchronized void assign(Socket socket) throws Exception {
    // 监听器线程给socket变量赋值,如果当前socket可用(即已经被赋过值还没被工作线程取走),则监听器线程进行等待
    while (available) {
        Logs.SERVER.info("{} wait assign socket : {}", Thread.currentThread().getName(), socket);
        this.wait();
    }
    // 若socket状态不可用,则监听器线程赋值成功;并将状态置为可用,因为此时socket已经有值,可以让工作线程来取
    this.socket = socket;
    available = true;
    // 上边赋值成功后,监听器线程通知在等待的工作线程可以来取socket了
    this.notify();
}

/**
 * 工作线程取出当前的socket.
 */
private synchronized Socket await() throws Exception {
    // 工作线程来取socket,如果当前socket不可用(即socket还没有被赋值),则工作线程进行等待
    while (!available) {
        Logs.SERVER.info("{} wait get socket", Thread.currentThread().getName());
        this.wait();
    }
    // socket可用,则工作线程取到socket;并将状态置为不可用,因为工作线程已经取走
    Socket socket = this.socket;
    available = false;
    // 工作线程通知监听器线程:现在socket对象已被取走,监听器线程可以再去给socket赋值了
    this.notify();
    return socket;
}

```

注意此处通知用的nofity，还一个notifyAll方法为什么不用呢？因为读线程只有一个、写线程也只有一个，所以用notify就够了，如果读写线程有多个，那么我们就得用notifyAll了；

## 问题

到这一版，我们已经实现监听器线程和工作线程的分离，使其各司其职，但是还存在着问题：

1. 请求依旧是串行化处理，因为即使监听器线程接收多个Socket，但是工作线程只有一个，工作线程处理一个Socket，监听器线程才能放下一个，其实监听器线程会出现忙等；

## 感想

遇到问题、解决问题，其实一些事情并没有想像中的复杂，当我们不了解的时候，只因我们没有遇到那个场景；

纸上得来终觉浅，绝知此事要躬行，动动手，一行一行自己写出来，去debug，便了解的更透彻；
