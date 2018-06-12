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

这里要讲的不是硅谷的那个风投公司Y-Combinator，而是讲函数式编程中的一个概念；

说起Y Combinator，不得不提不动点，什么是不动点，比如一个求平方的函数

```
f(x) = x^2
那么当x为0和1的时候，就叫这个函数的不动点，即f(x) = x;
```

在函数式编程中，函数f的不动点是另外一个函数`z`即`f(z) = z`，Y Combinator是一个特殊的函数，它能返回函数`f`的不动点，`Y(f) = f(Y(f)) = f(z) = z`；

## 有什么用

没用，业务开发中这东西确实没用；

说实话，我现在也不知道这个东西有啥用，平时该写`if else`还是要写`if else`，该写`for each`还写`for each`；

理论上讲，Y Combinator是要实现匿名函数自身的递归调用，也就是在没有函数名的前提下函数自己调用自己；

## 如何推导

此处我就以[《The Little Schemer》](https://book.douban.com/subject/27080946/)中的推导过程来说明，因为作者这种苏式问题式的讲解已经十分通俗易懂，有兴趣的可以直接看这本书；

```

;Y Combinator

;声明一个叫array_length的函数，入参为列表l

(define array_length
  (lambda (l)
    (cond
      ((null? l) 0)
      (else (+ 1 (array_length (cdr l)))))))

; 计算空列表
(array_length '())

; 计算有三个的列表
(array_length '(a b c))

; 将array_length的定义去掉,变成一个匿名函数
(lambda (l)
    (cond
      ((null? l) 0)
      (else (+ 1 (array_length (cdr l))))))

;此时如果入参是空列表,返回0,如果非空列表则报错,因为我们还没有声明array_length函数;
((lambda (l)
    (cond
      ((null? l) 0)
      (else (+ 1 (? (cdr l)))))) '())

;那么我们匿名函数可以这么写
(lambda (l)
  (cond
    ((null? l) 0)
    (else (+ 1 (? (cdr l))))))

;此时我们将?用上边的匿名函数来替换,此时传为长度为1的列表,结果为1,即(+ 1 0),因为里边的那层匿名函数计算结果为0
(lambda (l)
  (cond
    ((null? l) 0)
    (else (+ 1
             ((lambda (l)
                (cond
                  ((null? l) 0)
                  (else (+ 1 (? (cdr l))))))
              (cdr l))))))


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

;抽象,可计算长度为0的空列表
((lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 (len (cdr l)))))))
 +)

;可计算长度为1的列表
((lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 (len (cdr l)))))))
 ((lambda (len)
    (lambda (l)
      (cond
        ((null? l) 0)
        (else (+ 1 (len (cdr l)))))))
  length))

;可计算长度为2的列表
((lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 (len (cdr l)))))))
 ((lambda (len)
    (lambda (l)
      (cond
        ((null? l) 0)
        (else (+ 1 (len (cdr l)))))))
  ((lambda (len)
     (lambda (l)
       (cond
         ((null? l) 0)
         (else (+ 1 (len (cdr l)))))))
   length)))

;无非是抽象出来的函数自己调用自己,再次抽象,将调用者主体抽象出来.可计算长度为0的空列表.
;(并且最后一层传给mk-length的函数根本用不上,所以传任意参数都可以,我们将length替换为+依旧可以这执行)
((lambda (mk-length)
   (mk-length +))
 (lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 (len (cdr l))))))))

;可计算长度为1的列表
((lambda (mk-length)
   (mk-length
    (mk-length +)))
 (lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 (len (cdr l))))))))

;可计算长度为2的列表
((lambda (mk-length)
   (mk-length
    (mk-length
     (mk-length +))))
 (lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 (len (cdr l))))))))

;上边那么多层嵌套无非是为了一层层递归直至列表长度为0,既然最后一层mk-length传递的参数可以是任意的,我们将它自己传递过去
;同样如上边,这个只能执行空列表,大于0的列表会报错,为什么呢？因为此时的参数len对应的入参mk-length,mk-length要接收的参数是函数而不是列表(cdr l)
((lambda (mk-length)
   (mk-length mk-length))
 (lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 (len (cdr l))))))))

;所以为了支持长度为1的列表,我们给len传一个函数,可以是任意的,比如+
((lambda (mk-length)
   (mk-length mk-length))
 (lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 ((len +) (cdr l))))))))

;此时将+替换为len自己,我发现可以执行长度的列表,那么此时这个函数就可以计算任意长度的列表了
;函数执行时不断递归,直至(cdr l)为空列表
((lambda (mk-length)
   (mk-length mk-length))
 (lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 ((len len) (cdr l))))))))

;不过还有一个问题,函数此时不再是这种形式(len (cdr l),而是(len len)这种,我们再进行抽象,将(len len)抽象出来
((lambda (mk-length)
   (mk-length mk-length))
 (lambda (len-two)
   ((lambda (len)
      (lambda (l)
        (cond
          ((null? l) 0)
          (else (+ 1 (len (cdr l)))))))
    (lambda (l)
      ((len-two len-two) l)))))

;再将中间函数主体也抽象出来
((lambda (len)
   ((lambda (mk-length)
      (mk-length mk-length))
    (lambda (len-two)
      (len (lambda (l)
            ((len-two len-two) l))))))
 (lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 (len (cdr l))))))))

;再将变量名简化一下
((lambda (f)
   ((lambda (x) (x x))
    (lambda (x)
      (f (lambda (l)
            ((x x) l))))))
 (lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 (len (cdr l))))))))

;到这里Y Combinator也出来了
(define Y
 (lambda (f)
   ((lambda (x) (x x))
    (lambda (x)
      (f (lambda (l)
            ((x x) l)))))))

;执行结果为4
((Y
 (lambda (len)
   (lambda (l)
     (cond
       ((null? l) 0)
       (else (+ 1 (len (cdr l)))))))) '(a b c d))

```

## 参考

[函数式编程的 Y Combinator 有哪些实用价值](https://www.zhihu.com/question/20115649)

[scheme下的停机问题和Y组合子](http://www.cppblog.com/huaxiazhihuo/archive/2013/07/11/201689.html)

[learning scheme](https://github.com/martin-liu/learning/tree/master/scheme)

