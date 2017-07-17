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

## 概述
本文主要以实际中的例子来讲责任链模式；希望以实际当中的一些例子来看别人是如何使用的，对我们平时开发有什么帮助。

## 讲个故事
上小学的时候，班里经常会有同学说，帮我给个纸条给那谁谁谁，比如坐第一排的白居易同学新写了一首诗，要传纸条给坐在第五排的刘禹锡同学炫耀一下，于是通过中间这一个个同学传递，便是一个典型的责任链模式;

另一个故事，在《who build america》纪录片中，石油大王Rockefeller为打破石油运输被铁路大亨Vanderbilt垄断的局面，出资修建布满全国的石油管道，这一节节的石油管道，也是一个典型的责任链模式；

由此看来，从小学生到商业巨头，都在使用责任链模式;

## 是什么
* 不就是烤串么？
* 不就是水管么？
* 不就是链表么？
* 不就是工作流么？
* ...

## 有什么用

## 举例

### Unix

### Servlet

### Tomcat

## 实例业务系统中应用

## 总结

## 引用

### Servlet
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
