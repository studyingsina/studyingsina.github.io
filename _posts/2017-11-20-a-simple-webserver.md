---
layout: post
title: "Java多线程系列-先写一个简单WebServer"
description: "Java 多线程 线程池"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![Metrics](/css/pics/2017-11-20-webserver-tou.jpg)

## 背景

想通过实际场景来介绍下多线程的应用，思来想去，就以开发一个Demo版的Web容器为例子来讲一下吧，这里面会用到线程池、线程通信、线程同步等知识点，能把多线程用到的知识点给贯穿起来；

## 需求

先来看看我们的需求背景，我们要开发一个Web容器，满足以下功能：

1. 支持静态文件的访问：比如html jpg js等；
2. 支持表态文件的并发访问：因为我们一个界面上往往有很多资源，比如一个网页除了index.html文件之外、网页内还会有引入一些js、图片等资源，如果请求都是串行处理（即请求完index.html，再一个一个请求js文件、图片文件），用户浏览体验差；
3. 我们要支持HTTP/1.1协议；
4. 当然，因为是Demo，只是为了学习多线程编程，我们不过多的考虑性能、扩展性；

需求场景如下图：

![WebServer](/css/pics/2017-11-20-webserver.png)

## 功能开发

### 单线程版
了解了以上需求，我们先进行简单的功能开发，那我们先回顾下如何实现一个简单的WebServer；

当用户从浏览器输入一个地址，比如：<http://localhost:8080/>，其实是请求到我们WebServer的机器并且连接上8080端口，告诉WebServer，我们请求方法为GET、请求路径是/（根路径），告诉WebServer，如果你能找到这个请求对应的资源，那么按HTTP协议格式给我返回，如果你找不到，也按照HTTP协议格式给我返回，明白了WebServer接收一个浏览器请求的过程，那么我们的功能实现起来就简单的多了，如下：

1. 服务端启动8080端口，并一直监听；
2. 监听到有客户端（比如浏览器）要请求<http://localhost:8080/>，那么TCP三次握手，建立连接；
3. 建立连接后，读取此次连接客户端传来的内容（其实就是解析网络字节流并按HTTP协议去解析）；
4. 解析到请求路径（比如此处是根路径），那么去根路径下找资源（比如此处是index.html文件）；
5. 找到资源后，再通过网络流将内容输出，当然，还是按照HTTP协议去输出，这样客户端（浏览器）就能正常渲染、显示网页内容；

调用过程如下图：

![WebServer-Process](/css/pics/2017-11-20-webserver-process.png)

### 代码实现
其实看明白上边的处理流程，剩下的便是用Java提供的API去实现了，比如简单，贴一段主要代码，完整代码后边有链接：

```Java

    /**
     * 接收客户端的Socket,解析输入字节流,并返回结果.
     * @throws Exception
     */
    private void process() throws Exception {
        // 2. 监听到有客户端（比如浏览器）要请求http://localhost:8080/，那么建议连接，TCP三次握手；
        Socket socket = ss.accept();
        InputStream is = socket.getInputStream();
        OutputStream os = socket.getOutputStream();
        BufferedReader reader = new BufferedReader(new InputStreamReader(is));

        /**
         * 3. 建立连接后，读取此次连接客户端传来的内容（其实就是解析网络字节流并按HTTP协议去解析）；
         * GET /dir1/dir2/file.html HTTP/1.1
         */
        String requestLine = reader.readLine();
        Logs.SERVER.info("requestLine is : {}", requestLine);
        if (requestLine == null || requestLine.length() < 1) {
            Logs.SERVER.error("could not read request");
            return;
        }

        String[] tokens = requestLine.split(" ");
        String method = tokens[0];
        String fileName = tokens[1];
        File requestedFile = docRoot;

        String[] paths = fileName.split("/");
        for (String path : paths) {
            requestedFile = new File(requestedFile, path);
        }
        if (requestedFile.exists() && requestedFile.isDirectory()) {
            requestedFile = new File(requestedFile, "index.html");
        }

        BufferedOutputStream bos = new BufferedOutputStream(os);
        // 4. 解析到请求路径（比如此处是根路径），那么去根路径下找资源（比如此处是index.html文件）；
        if (requestedFile.exists()) {
            Logs.SERVER.info("return 200 ok");
            long length = requestedFile.length();
            BufferedInputStream bis = new BufferedInputStream(new FileInputStream(requestedFile));
            String contentType = URLConnection.guessContentTypeFromStream(bis);
            byte[] headerBytes = createHeaderBytes("HTTP/1.1 200 OK", length, contentType);
            bos.write(headerBytes);

            // 5. 找到资源后，再通过网络流将内容输出，当然，还是按照HTTP协议去输出，这样客户端（浏览器）就能正常渲染、显示网页内容；
            byte[] buf = new byte[2000];
            int blockLen;
            while ((blockLen = bis.read(buf)) != -1) {
                bos.write(buf, 0, blockLen);
            }
            bis.close();
        } else {
            Logs.SERVER.info("return 404 not found");
            byte[] headerBytes = createHeaderBytes("HTTP/1.0 404 Not Found", -1, null);
            bos.write(headerBytes);
        }
        bos.flush();
        socket.close();
    }

    /**
     * 生成HTTP Response头.
     *
     * @param content
     * @param length
     * @param contentType
     * @return
     */
    private byte[] createHeaderBytes(String content, long length, String contentType) throws Exception {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(baos));
        bw.write(content + "\r\n");
        if (length > 0) {
            bw.write("Content-Length: " + length + "\r\n");
        }
        if (contentType != null) {
            bw.write("Content-Type: " + contentType + "\r\n");
        }
        bw.write("\r\n");
        bw.flush();
        byte[] data = baos.toByteArray();
        bw.close();
        return data;
    }

```

完整代码实现：[BootstrapV1.java](https://github.com/studyingsina/concurrency-programming-demo/blob/master/src/main/java/com/studying/concurrency/v1/BootstrapV1.java)

## 问题
我们实现了一版WebServer，但是有什么问题呢？

1. 我们让main线程去监听8080端口，并且main线程去查询资源、返回结果；
2. 因为只main一个线程处理所有逻辑，就导致了我们首页的5个请求（一个index.html界面和4个图片资源）是串行的处理，即先处理了index.html文件，然后依次处理每个图片，如果同时有多个人打开浏览器请求，那么所有的请求也都是串行的；

## 感想
这一版仅实现功能，算是第一次迭代，有了一个可用的版本；接下来，我们会进行多次迭代，一步步的完善；其实实际业务开发中，不也是这样么，没有一开始就设计完美无缺的系统、都是多次迭代、运维出来的系统。
