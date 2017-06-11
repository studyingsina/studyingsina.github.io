---
layout: post
title: "设计模式-责任链之抛砖引玉(未完)"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![未来](/css/pics/2017-book-list.jpg)

## 讲个故事
《天龙八部》中王语嫣，熟读各派武学秘笈，简直是一座活的藏经阁，但是她的实际战斗力为0，为什么？因为没有实践；

同样，我们写代码除了要看一些理论、概念的书，还得有大量的实践；

## 内容概述
合适的地方用合适的工具，设计模式是我们编程的工具;

今天就我们不讲所有设计模式，因为太多，只讲一个责任链模式；

什么？责任链模式还需要花篇幅介绍？

这次不介绍概念，因为讲概念的书汗牛充栋，主要讲在实践中一些责任链的用法，看看别人是在什么场景下用的，用的好不好、合理不合理？最后再讲一个我在实际项目当中的应用（不一定用的就合适，权当例子参考）；

## 介绍
* 不就是烤串么？
* 不就是水管么？
* 不就是链表么？
* 不就是工作流么？
* ...

## 实现方式
实践中责任链模式有好多实现，其中不乏一些高手写的实现，供我们学习、借鉴；

### Filter
```Java
/**
 * 创建并下载文件
 * @param  {String} fileName 文件名
 * @param  {String} content  文件内容
 */
function createAndDownloadFile(fileName, content) {
    var aTag = document.createElement('a');
    var blob = new Blob([content]);
    aTag.download = fileName;
    aTag.href = URL.createObjectURL(blob);
    aTag.click();
    URL.revokeObjectURL(blob);
}
```

### Pipeline

### 实现项目当中的应用 

## 总结

