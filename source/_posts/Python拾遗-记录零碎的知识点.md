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

# Python 使用virtualenv 管理虚拟环境
virtualenv和virtualenvwrapper介绍、安装和使用


# python ensure_ascii

```
>>> import json
>>> print json.dumps('中国')
"\u4e2d\u56fd"
```
输出的会是
'中国' 中的ascii 字符码，而不是真正的中文。
这是因为json.dumps
序列化时对中文默认使用的ascii编码.想输出真正的中文需要指定ensure_ascii=False：
```
dumps(obj, skipkeys=False, ensure_ascii=True, check_circular=True,
        allow_nan=True, cls=None, indent=None, separators=None,
        encoding='utf-8', default=None, sort_keys=False, **kw):
```
ensure_ascii 默认是　True 对
```
def py_encode_basestring_ascii(s):
    """Return an ASCII-only JSON representation of a Python string

    """
    if isinstance(s, str) and HAS_UTF8.search(s) is not None:
        s = s.decode('utf-8')
    def replace(match):
        s = match.group(0)
        try:
            return ESCAPE_DCT[s]
        except KeyError:
            n = ord(s)
            if n < 0x10000:
                return '\\u{0:04x}'.format(n)
                #return '\\u%04x' % (n,)
            else:
                # surrogate pair
                n -= 0x10000
                s1 = 0xd800 | ((n >> 10) & 0x3ff)
                s2 = 0xdc00 | (n & 0x3ff)
                return '\\u{0:04x}\\u{1:04x}'.format(s1, s2)
                #return '\\u%04x\\u%04x' % (s1, s2)
    return '"' + str(ESCAPE_ASCII.sub(replace, s)) + '"'
```
```
>>> import json
>>> print json.dumps('中国')
"\u4e2d\u56fd"
>>> print json.dumps('中国',ensure_ascii=False)
"中国"
>>>
```
