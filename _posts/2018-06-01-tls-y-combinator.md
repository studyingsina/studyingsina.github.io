---
layout: post
title: "the little schemer - Y combinator"
categories: Tech
tags: scheme
---

* content
{:toc}

![Y Combinator](/css/pics/2018-06-01-tls-y-combinator.jpg)


## 是什么


## 有什么用

说实话，我现在也不知道这个东西有啥用，在实际的业务的开发中该写if else还是要写if else，该用for循环还用for循环；

从理论上讨论，Y Combinator是要实现匿名函数自身的递归调用，也就是在没有函数名的前提下函数自己调用自己；

## 如何推导

此处我就以[《The Little Schemer》](https://book.douban.com/subject/27080946/)中的推导过程来说明，有兴趣的可以直接看这本书；

我们先来写一个简单的函数：计算数组的长度：

```
;声明一个叫array_length的函数，入参为数组l

(define array_length
  (lambda (l)
    (cond
      ((null? l) 0)
      (else (+ 1 (array_length (cdr l)))))))

; 计算空数组,返回0
(array_length '())

; 计算有三个的数组,返回3
(array_length '(a b c))

```

这里我们定义了函数array_length，当入参l为空的时候返回数组长度为0，当参数l不为空的时候递归调用自身；

但是在lambda演算中，没有函数名，我们将函数名字去掉：

```
; 将array_length的定义去掉,变成一个匿名函数
(lambda (l)
    (cond
      ((null? l) 0)
      (else (+ 1 (array_length (cdr l))))))

;此时如果入参是空列表,返回0,如果非空列表则报错,因为我们还没有声明array_length函数;

```

其实我们心里可以称这个匿名函数为?，毕竟他只能计算长度为0的空列表；

问题来了，如果我们想计算长度小于等于1的列表，该如何写这个匿名函数呢？可以将上边占位符?再用匿名本身去替换掉：

```

;此时我们将?用上边的匿名函数来替换
(lambda (l)
  (cond
    ((null? l) 0)
    (else (+ 1
             ((lambda (l)
                (cond
                  ((null? l) 0)
                  (else (+ 1 (? (cdr l))))))
              (cdr l))))))

```

此时我们心里可以称这个匿名函数为array_length1，毕竟他只能计算长度小于等于1的列表；同样，当我们想计算长度小于等于2的列表时，可以再多嵌套一层：

```

;计算长度小于等于2列表,我们需要再多嵌套一层
(lambda (l)
  (cond
    ((null? l) 0)
    (else (+ 1
             ((lambda (l)
                (cond
                  ((null? l) 0)
                  (else (+ 1
                           ((lambda (l)
                              (cond
                                ((null? l) 0)
                                (else (+ 1 (? (cdr l))))))
                            (cdr l))))))
              (cdr l))))))

```

此时我们心里可以称这个函数为array_length2，毕竟他只能计算长度小于等于2的列表，那么如果我们想计算任意长度的列表，可以一些这么嵌套下去直至无穷大；还记得学习scheme的时候，书中有十大原则么，其中一个是：把相同的功能抽象出来-没错、就是抽象；



## 函数式语言


## 参考

[函数式编程的 Y Combinator 有哪些实用价值](https://www.zhihu.com/question/20115649)


[scheme下的停机问题和Y组合子](http://www.cppblog.com/huaxiazhihuo/archive/2013/07/11/201689.html)

