---
layout: post
title: "读《SRE:Google运维解密》"
categories: 
- Reading
tags:
- Reading
---

* content
{:toc}

### 为什么看这本书
一同事推荐这本书，并且同事将书带公司了，看一看。[《SRE:Google运维解密》](https://book.douban.com/subject/26875239/)。

### 读后感
评分7，值得一看，翻译是google运维工程师一线人员，所以本书翻译读起来还算流畅，没有那么生硬;

前几单讲了一些google运维的一个理念、或者叫方法论，比较认同的地方：

* 尽量避免重复工作，重复的事情是不是可以通过写些工具来处理，工具越积越多、效率提升就越多;

* 系统达到一定的可用性后，不追求100%的完美，因为99.999%和100%对用户来说差距并不大，但是追求100%要比99.999%要付出多的多，其实还是一个投入产出比的问题；

* 接上边一条，Chubby是高可用的，在满足一定可用性的情况下，会刻意的让Chubby不可用会儿，否则业务会认为Chubby百分百可靠，形成过度依赖，当Chubby真的出问题时，业务影响过多，其实对业务来说是提高危机意识，自己平时就做好业务降级；

* 运维，不仅仅是运维，要有50%的时候用来开发，SRE中的E便是engineer；

我们大部分人一般工作生涯中不会遇到像google这样大规模的公司，但是可以通过网络、书等一些途径去参考、借鉴这些公司的经验；其实更重要的是：别人在遇到这个问题时如何思考、如何处理的这个过程，更值得我们去学习。
