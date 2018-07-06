---
title: Python拾遗-记录零碎的知识点
meta: python
date: 2018-06-22 12:14:23
tags: Python
categories: Python
keywords: Python
---
# python Lambda

Lambda作用：
Python中，lambda函数也叫匿名函数，及即没有具体名称的函数，它允许快速定义单行函数，类似于C语言的宏，可以用在任何需要函数的地方。这区别于def定义的函数。

#### 核心：返回一个函数
举例：
```
g = lambda x : x ** 2
print g(3)
```
#### 定义规范 ：如何定义一个lambda表达式
```
lambda [arg1 [, agr2,.....argn]] : expression
```

#### lambda 作为参数传递 和 无参数lambda

```
def test_lambda(num):
  print(num())

test_lambda(lambda:2)
```

#### lambda与def的区别：
* 1）def创建的方法是有名称的，而lambda没有。
* 2）lambda会返回一个函数对象，但这个对象不会赋给一个标识符，而def则会把函数对象赋值给一个变量（函数名。
* 3）lambda只是一个表达式，而def则是一个语句。
* 4）lambda表达式” : “后面，只能有一个表达式，def则可以有多个。
* 5）像if或for或print等语句不能用于lambda中，def可以。
* 6）lambda一般用来定义简单的函数，而def可以定义复杂的函数。
* 7）lambda函数不能共享给别的程序调用，def可以。
