---
layout: post
title: "Hbase安装启动"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![hbase-install](/css/pics/2019-02-14-hbase_logo_with_orca_large.png)

## 背景

学习Hbase的使用，先本地安装下；

只介绍单机模式和伪分布式模式的安装，参考安装教程：[Quick Start](http://hbase.apache.org/book.html#quickstart "quickstart")；

## 单机模式

### 安装JDK

安装好JDK，此处不做过多介绍，我选择的JDK1.8，配置好JAVA_HOME环境变量；

### 下载HBase

选择一个版本下载[Apache Download Mirrors](https://www.apache.org/dyn/closer.lua/hbase/)，此处我下载的是hbase-2.0.4-bin.tar.gz；

```Java

tar xzvf hbase-2.0.4-bin.tar.gz
cd hbase-2.0.4

```

确保JAVA_HOME已经设置；

### 配置文件1

编辑配置文件conf/hbase-site.xml，将hbase.rootdir和hbase.zookeeper.property.dataDir更改为自己的目录；

默认不配置Hbase和ZK的路径，HBase会在/tmp目录下自动创建数据写入的文件夹，一些操作系统会在机器重启时自动清除/tmp目录下的内容，会导致数据丢失；

```XML

<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///home/testuser/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/home/testuser/zookeeper</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
    <description>
      Controls whether HBase will check for stream capabilities (hflush/hsync).

      Disable this if you intend to run on LocalFileSystem, denoted by a rootdir
      with the 'file://' scheme, but be mindful of the NOTE below.

      WARNING: Setting this to false blinds you to potential data loss and
      inconsistent system state in the event of process and/or node failures. If
      HBase is complaining of an inability to use hsync or hflush it's most
      likely not a false positive.
    </description>
  </property>
</configuration>

```

### 启动Hbase

执行bin/start-hbase.sh脚本启动Hbase，单机模式下HMaster、HRegionServer、ZK都在同一JVM中，执行jps命令可以看到HMaster进程，并且通过[HBase Web UI](http://localhost:16010/)可以看到HBase管理界面；

如下图：

![HBase-WebUI](/css/pics/2019-02-14-hbase_webui.png)

然后可以通过bin/hbase shell在命令行执行Hbase命令，此处不做过多介绍，下面介绍伪分布式模式的安装；

## 伪分布式模式


