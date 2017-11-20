---
layout: post
title: "Java 线程池"
description: "Java 多线程 线程池"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![Metrics](/css/pics/2017-11-13-multi-thread.jpg)

# 背景

想通过实际场景来介绍下多线程的应用，思来想去，就以开发一个Demo版的Web容器为例子来讲一下吧，这里面会用到线程池、线程通信、线程同步等知识点，能把多线程用到的知识点给贯穿起来；

# 需求

我们要开发一个Web容器，满足以下功能：

1. 支持静态文件的访问：比如html jpg js等；
2. 支持表态文件的并发访问：因为我们一个界面上往往有很多资源，比如网站首页除了index.html之外、还会有一些js、图片等资源，如果请求都是串行处理（即请求完index.html，再一个一个请求js文件、图片文件），用户浏览体验差；
3. 我们要支持HTTP/1.1协议；

# 功能开发

## 单线程版

## 多线程版

