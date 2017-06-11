---
layout: post
title: "设计模式-责任链之抛砖引玉"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![未来](/css/pics/2017-book-list.jpg)

## 内容概述

## 介绍

## 实现方式

### Filter
```js
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

### Tomcat

### 实现项目当中的应用 

## 总结

