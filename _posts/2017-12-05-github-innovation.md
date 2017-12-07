---
layout: post
title: "Github-没有做不到-只有想不到"
description: "Github 玩法 创新 创造力"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![Github](/css/pics/2017-12-05-github.png)

## 背景

体验过Github的几个产品，感觉开发Github的这批人真的很富有想像力；原来一个托管代码的网站还能这么玩：没有做不到、只有想不到；

所以今天我来谈谈Github的创新；

## Github能做什么

提起Github，可能大家第一印象便是：

    1. 不就一个托管代码网站么；
    2. 顺便存储些文件、图片，还能当一个网盘用；
    3. 可以在上面找些开源工具来用；
    4. 不注册一个Github账号，都不好意思跟别人打招呼；
    5. 程序猿交友网站（同性）

对，上边理解都有一定道理；要说Github的创新，我们来看看Github给我们提供了哪些产品？

    1. 写代码、托管代码，这点不用多说，但是Github一出场就能远甩SourceForge几条街，仅仅是因为它支持Git这个工具么？
    2. 写博客，什么，这不是WordPress这类软件做的事情么？你竟然用Github写博客，不好意思，Github真的可以，并且能让你像写代码一样写博客，让博客回归写作本质；
    3. 代码片段，这是什么东东？对不起，它有个正式的名字叫：Gist，我们稍后会介绍；
    4. 编辑器，当你还在讨论神用的编辑器和编辑器之神的时候，Github告诉我们，其实你还有另一种选择，Sublime？No No No，虽然它长的很像Subline，它有正式名字叫：Atom；
    5. 电子书，这TM不是epub、mobi之流的特长么，你敢说Github也支持？不错，Gitbook，写电子书，为你而来；
    6. 工具市场，什么？Github要卖东西，不错，如果你需要一些工具，可以直接到市场里去买，对于一些小创公司来说，节省成本，正如Github MarketPlace宣传视频里所说那样；
    7. Bug追踪，你是在逗我么？我们公司明明有用开源的BugFree软件，且慢，先来看看Github的Issues；
    8. StackOverFlow，神经病吧，这不是我们程序猿写代码的灵感源泉么？Github也有这功能，有没有我不知道，总之以前技术难题都去stackoverflow上查，现在发现有些难题得去Github上查；
    9. 招聘，你是想逼疯HR么，Github说：我也不想的，只是有HR这么干，从Github上找候选人，怪我喽？
    10. 在线简历，行行行，I服了You，你到底还有哪些事情不能做？
    11. . . . . . .

别着急，且听我慢慢给你道来。

###  代码托管

曾几何时，代码托管不是SourceForge、GoogleCode这类产品的天下么，怎么突然出来一个Github，把市场搅的天翻地覆，如果仅仅做为支持Git工具的角度来看，Github还真不比前边那些网站有优势，毕竟人家要么家大业大、要么起步早积累厚，但是Github不跟你们拼这些，Github说：我除了长像小清新、支持版本控制工具Git外，我还支持PR、Fork等功能，我还是代码网站中的微博、我能让程序猿们Follow他们喜欢的猿、我让用户有粘性，你要硬说我是程序猿交友网站，我也不反对；当你们还在苦苦力拼谁支持的免费空间更大、更长的时候，我玩的是社交，懂么？要想有粘性、得靠互动、得有社交；这时候是来看一下这张网站流传的图了：

![Github-SS](/css/pics/2017-12-05-github-ss.jpg)

在代码托管的领域里，Github说：我先行一步，你们这些老古董们慢慢追。

### 写博客

既然代码能托管，那么文章为啥就不能托管呢？能、能、能，Github Pages就是干这个的；你可以用你喜欢的任一款编辑器写完文章，Push到Github便可，此时文章便可访问。

什么？这么简单？难道不需要租得VPS、建数据库、搭Nginx、买域名？用WordPress五分钟建站、或者直接上AWS套餐不也也简单么？如果你觉得网站版富文本编辑器好用，那我也无话可说。

用过多款建站工具之后，Github Pages这类静态模板网站，真的是专为开发人员而设；不错，本站正是用Github Pages，Fork一个你喜欢的代码仓库，便可以开始写博客了。

当然，如果非开发人员，只要你会Git几个命令，上手也很简单；

### 代码片段

如果你平时需要将一些片段代码保存、留待后用、查看，那么试试Gist，比如这个场景：开发人员在写文章的时候，要在文章中嵌入一些代码片段，用Gist就恰好，看下边这段代码：

如果你所在网络不能访问Gist，看截图：

![Gist](/css/pics/2017-12-06-gist.jpg)

### 编辑器

如果你用过Sublime，那么看到Atom，你会觉得很亲切；对js、Node同学是另一种不错的选择；畅想一下，加上Github的托管功能，Atom是不是就一个Evenote、云笔记？另加上强大的插件功能、原来的一些单体应用比如Mou是不是可以退休了？

Atom是想发展成程序猿日常工具的全家桶么？

![Atom](/css/pics/2017-12-07-atom.jpg)

### 电子书

如果你经常看技术文章，那么对Gitbook形式的电子书一定不陌生；也有很多开源项目的文档就用Gitbook来写，比如TensorFlow；

当然，你如果想用Gitbook写日记、写博客，也没人反对；

### 工具市场

工具市场有什么用呢？直接上官方宣传视频，一眼明了；

[![MarketPlace](/css/pics/2017-12-07-marketplace.jpg)](https://www.bilibili.com/video/av11116892)

现在市场里已经有Travis CI、Sentry等工具可用，这完全是模仿App Store的套路来的；

### 问题追踪

如果你有一个开源项目，别人有问题，可以用Issues给你提，是像Jira Task？还是像BugFree？还是像论坛留言板？你说像什么就像什么吧？反正用着挺爽；

如果大家都针对相应的技术问题用Issues讨论，那么StackOverFlow是不是感觉有点小危机？

如果一个人在Issues回答的问题质量又高、又多，是不是可以称得上大V？是不是也可以开一堂Github Live？

### 其它

用Github做一张简历？[Resume](http://resume.github.io/)

HR在上面筛选简历？

## 体会

能把一个代码托管网站玩出这么多花样，你能想得到么？

两岸猿声啼不住, 轻舟已过万重山.