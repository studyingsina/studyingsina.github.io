---
layout: post
title: "如何才叫了解一个业务系统"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![未来](/css/pics/2018-04-17-hold-a-project.jpeg)

## 背景
工作当中会接手一些别人交接给自己的业务系统，排查问题时经常需要去查看业务日志，但是往往业务日志不全，这就影响了定位问题的效率，然后开始完善业务日志、再上线、再重现问题，其实想想，业务系统打些日志，这不应该是很平常的事情么，为什么一些工作几年的同学都做不到呢？

难道只开发、不维护？谁又能保证开发的功能上线之后没有问题？通过这件事，我在想，作为一名开发人员，如何才能叫了解一个业务系统呢？

## 业务目标

要完成业务提的需求、理解业务；要了解系统解决的是什么问题，此处可以来个一句话话描述，本系统在公司各业务中处于一个什么位置？系统是给谁用的？再之后了解这个系统提供的有哪些功能？系统上下游都是谁？系统要保证SLA是多少？系统哪些功能是核心功能、哪些是非核心功能？

## 定位问题
出现问题要快速定位、并解决；

定准问题工具，日志 脚本 诊断工具；

日志这个事情，在学编程的过程中，没有人会告诉我们打日志，但是工作中，如何打日志却很能看出一个人工程素质（经验），如果平时有排查问题的情况，多多少都离不开日志，软件是有生命周期的，开发阶段只占一小部分，更长的时间我们是在维护，想想很多Java教程为了简单，都用System.out.print来演示，其实对于新手存在误导，如果一个入职的新人写代码，里面充斥着各种System.out.print或一行日志都没打，那很可能实际工作经验没多少；当然，对于一些QPS大的业务，线上实际不会打太多日志，但是却可以通过日志级别进行调整、采样上报；

## 熟悉代码实现
熟悉系统架构，代码逻辑实现，是计算型应用、存储型应用？定时任务何执行？是否存在单点问题？是否可水平扩展？是否存在SQL注入、被爬等安全风险？用户敏感信息是否加密？

## 资源占用
系统部署在实体机？虚拟机？容器中？系统占用多少进程、多少线程、最大线程数多少？占用内存、磁盘、带宽多少？是计算型应用、还是存储型？占用了哪些端口？外部依赖有哪些？哪些是核心依赖、哪些是非核心依赖？数据库容量多少、每天增量多少、数据库读写QPS是多少？缓存多少？数据库、缓存的部署架构是什么：主从？高可用怎么保证？日常流量分布、高峰低谷是什么时候？压测上限、单机承载上限是多少？节假日是否需要扩容？

## 人
系统需求量多少？季度规划？半年规划？目前有多少资源（前后端、PM、QA等资源）、各人员能力水平如何、能做多少事情？未来对系统的期望、做到什么程度？

## 总结
暂时想到哪就写到，没有很好总结；
