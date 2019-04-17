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

```shell

tar xzvf hbase-2.0.4-bin.tar.gz
cd hbase-2.0.4

```

确保JAVA_HOME已经设置；

### 配置文件1

编辑配置文件conf/hbase-site.xml，将hbase.rootdir和hbase.zookeeper.property.dataDir更改为自己的目录；

默认不配置Hbase和ZK的路径，HBase会在/tmp目录下自动创建数据写入的文件夹，一些操作系统会在机器重启时自动清除/tmp目录下的内容，会导致数据丢失；

```shell

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
  </property>
</configuration>

```

### 启动Hbase

执行bin/start-hbase.sh脚本启动Hbase，单机模式下HMaster、HRegionServer、ZK都在同一JVM中，执行jps命令可以看到HMaster进程，并且通过[HBase Web UI](http://localhost:16010/)可以看到HBase管理界面；

如下图：

![HBase-WebUI](/css/pics/2019-02-14-hbase_webui.png)

然后可以通过bin/hbase shell在命令行执行Hbase命令，此处不做过多介绍，下面介绍伪分布式模式的安装；

## 伪分布式模式


### zookeeper

下载并启动zookeeper，[参见](https://zookeeper.apache.org/doc/r3.4.13/zookeeperStarted.html)

### hadoop

下载并安装hadoop，[参见](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)

此处我用的是hadoop伪分布式模式；

启动成功后，可以访问[Hadoop Web UI](http://localhost:50070/dfshealth.html#tab-overview)

通过jps命令可以看到nameNode和dataNode线程：

```shell

➜  hadoop-2.7.7 jps
70433 Jps
70176 DataNode
70087 NameNode
69274 QuorumPeerMain
70285 SecondaryNameNode

```

### hbase

修改配置文件conf/hbase-site.xml:

```shell

<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://localhost:9000/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
</configuration>

```

执行hbase启动命令:bin/start-hbase.sh，启动成功执行jps命令可以看到HMaster和HRegionServer线程，如下：

```shell

➜  hbase-2.0.4 jps
70176 DataNode
70707 Jps
70547 HQuorumPeer
70087 NameNode
70600 HMaster
69274 QuorumPeerMain
70669 HRegionServer
70285 SecondaryNameNode

```

此时也可以通过[Hbase Web UI](http://localhost:16010/master-status)查看；

到hadoop安装目录执行./bin/hadoop fs -ls /hbase 命令，可以看到HDFS中存在/hbase目录，如下：

```shell
➜  hadoop-2.7.7 ./bin/hadoop fs -ls /hbase
19/03/21 21:31:26 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 13 items
drwxr-xr-x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/.hbck
drwxr-xr-x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/.tmp
drwxr-xr-x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/MasterProcWALs
drwxr-xr-x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/WALs
drwxr-xr-x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/archive
drwxr-xr-x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/corrupt
drwxr-xr-x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/data
drwxr-xr-x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/hbase
-rw-r--r--   3 junweizhang supergroup         42 2019-03-21 21:30 /hbase/hbase.id
-rw-r--r--   3 junweizhang supergroup          7 2019-03-21 21:30 /hbase/hbase.version
drwxr-xr-x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/mobdir
drwxr-xr-x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/oldWALs
drwx--x--x   - junweizhang supergroup          0 2019-03-21 21:30 /hbase/staging

```

然后便可以通过hbase shell执行hbase命令了，[参见](http://hbase.apache.org/book.html#shell_exercises)

### 解决什么问题
大数据实时读写；官网说数据上亿或十亿，其实目前对于MySQL来说单表上亿、单列索引


### 适用场景

