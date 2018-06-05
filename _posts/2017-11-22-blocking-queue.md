---
layout: post
title: "Java多线程系列-缓解忙等问题"
description: "Java 多线程 线程池"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![Queue](/css/pics/2017-11-22-cache-queue.jpg)

## 背景

接着上一篇[《Java多线程系列-开始给工作线程减压》](http://www.longtask.net/2017/11/21/reduce-worker/)我们提出的问题，监听器线程会一直等待工作线程处理完一个socket后，再去接收下一个socket，为了缓解这个问题，我们引入了缓冲队列，简单说：就是将上一节当中的共享变量WebServer.socket换成一个队列，这样我们监听器接收完socket往队列里放、不再直接依赖工作线程处理完，貌似还真的是计算机中的所有问题都可以通过引入一个中间层去解决；

打个比方：我们的餐馆有了一个前台接待（监听器线程）、一个厨师兼职传菜（工作线程），只有一个位置（共享变量），当前台接待一名用户进来后，用户将这个位置占了，厨师也开始忙活了，此时前台又接待了一个新用户，只能告诉新用户：只有一个位置已经有人了，麻烦您等会；现在我们想同时接待多个用户，采用的一个办法就是添加位置（队列），这样前台接待一位用户便可以将其安排在空闲的位置了（队列）；

## 需求

1. 通过引入一个阻塞队列，来缓解监听器线程忙等的问题；

## 功能开发

这一版我们并没引入线程新的知识点，只是自己动手写了一个简单的阻塞队列；

### 阻塞队列

一个简单的阻塞队列:先进先出,线程安全,不支持扩容,用数组实现.

```java

/**
 * 一个简单的阻塞队列(先进先出),线程安全,不支持扩容,用数组实现.
 */
public class SimpleQueue<E> {

    // 元素数据
    private Object[] items;

    // 队列容量
    private int capacity;

    // 队列头索引
    private int putIndex;

    // 队列尾索引
    private int takeIndex;

    // 队列当前元素个数
    private int size;

    public SimpleQueue(int cap) {
        this.capacity = cap;
        this.items = new Object[cap];
        this.size = 0;
        this.putIndex = 0;
        this.takeIndex = 0;
    }

    public synchronized void put(E e) throws InterruptedException {
        // 监听器线程往队列中放入socket,如果当前队列满了则监听器等待
        if (isFull()) {
            Logs.SERVER.info("{} wait put queue : {}", Thread.currentThread().getName(), e);
            wait();
        }
        // 若队列没满,则监听器线程往队列中放入socket;并且如果先前已经有工作线程在等待取数据,通知工作线程来取
        items[putIndex] = e;
        putIndex = (putIndex + 1) % capacity;
        size++;
        Logs.SERVER.info("queue isFull {}, isEmpty {}, capacity {}, size {}, takeIndex {}, putIndex {}", isFull(), isEmpty(), capacity, size, takeIndex, putIndex);
        notify();
    }

    public synchronized E take() throws InterruptedException {
        // 工作线程来取socket,如果当前队列为空,则工作线程进行等待
        if (isEmpty()) {
            Logs.SERVER.info("{} wait get socket", Thread.currentThread().getName());
            wait();
        }
        // 队列不为空,工作线程从队列中取出socket;并且如果先前有监听器线程在等待往队列中放数据,通知监听器线程放
        E e = (E) items[takeIndex];
        // 将已经取走的引用置为空,让GC可以回收
        items[takeIndex] = null;
        takeIndex = (takeIndex + 1) % capacity;
        size--;
        Logs.SERVER.info("queue isFull {}, isEmpty {}, capacity {}, size {}, takeIndex {}, putIndex {}", isFull(), isEmpty(), capacity, size, takeIndex, putIndex);
        notify();
        return e;
    }

    public synchronized boolean isFull() {
        return capacity == size;
    }

    public synchronized boolean isEmpty() {
        return size == 0;
    }

}

```
如果你所在的网络能访问Gist，这个格式可能更友好：

{% gist a3b4165eeb96b02e2814d664fdda6418 %}

完整代码实现：[WebServer.java](https://github.com/studyingsina/concurrency-programming-demo/blob/master/src/main/java/com/studying/concurrency/v4/WebServer.java)

### 知识点

因为我们的队列是线程安全的，所以WebServer.assign和await方法便无需再加synchronized的关键字了；

在队列中的通知方法依旧用的notify方法，为什么不用notifyAll方法呢？因为监听器线程（生产者线程）只有一个、工作线程（消费者线程）也只有一个；

还有个细微点要注意的就是：synchronized关键字上一节也用到了，但是和本节的synchronized关键字所表示的锁对象是不一样的，上一节WebServer拥有一个Socket变量，的锁对象是WebServer、由WebServer自己控制同步、阻塞，而本节使用了队列，锁对象是加在队列上的；

```java

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

```

## 问题

到这一版，我们已经缓解了监听器线程忙等问题，但是还存在问题：

1. 若监听器线程的接收速度要快于工作线程的速度，那么队列就会处理不及时，等队列满了之后还会存在监听器线程忙等；

## 感想

今天上班地铁上，看到一首诗，白居易的，人生时起时落，努力学会淡然：

《风雨晚泊》

苦竹林边芦苇丛，停舟一望思无穷。

青苔扑地连春雨，白浪掀天尽日风。

忽忽百年行欲半，茫茫万事坐成空。

此生飘荡何时定，一缕鸿毛天地中。
