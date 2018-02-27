---
title: Python冲刺系列-(Python装饰器)
meta: Python冲刺系列-(Python装饰器)
date: 2018-02-03 16:26:43
tags: Python装饰器
categories: Python系列
keywords: Python 装饰器
---
#### 啰嗦
面试一说到Python,不问一句装饰器,都显得自己不够专业.所以就在这里总结一下,在下对装饰器的理解.

#### 理解装饰器
要理解装饰器,要从函数开始.
##### 理解Python中的函数
在Python的世界里,也是一切皆对象.例如:list tuple dict func都是对象.
既然函数是一个对象,那么就是可以使用变量名进行引用.
```code 
def func(a,b):
    return a+b
```
就是:ref = func 这里区别下ref = func() **一个赋值的是函数对象,一个赋值的是调用函数后的返回值**

##### 理解闭包
>在python中闭包,也是一个对象,是一个将组成函数和这些语句执行环境打包在一起后的一个对象.
闭包最重要的使用价值在于：封存函数执行的上下文环境；
闭包在其捕捉的执行环境(def语句块所在上下文)中，也遵循LEGB规则逐层查找，直至找到符合要求的变量，或者抛出异常。

举例🌰:
```code
def caculate():
    alist = range(1,101)
    def lazy_sum():
        return reduce(lambda x,y:x+y,alist)
    return lazy_sum


sum = caculate()
print sum()
```
闭包对于封装一部分参数在某一时候调用 是一个典型的懒加载
这里讲 lazy_sum 函数,作为一个引用变量,分存在闭包的上下文环境中.

##### 装饰器的语法糖 @符号

```code
class A:
    @classmethod
    def method(cls):
        pass

class B:
    # Equivalent definition of a class method
    def method(cls):
        pass
    method = classmethod(method)
```
这两个 method 方法是等价的. 函数作为一个变量,被封存在闭包的上下文环境中.

##### 举个闭包的例子
```code
def checkParams(fn):
    def wrapper(a, b):
        if isinstance(a, (int, float)) and isinstance(b, (int, float)):
            return fn(a, b)
        print('variable a and b not be added')
        return
    return wrapper


@checkParams
def add(a,b):
    return a+b

num = add(1,'a')
```
##### 带参数的装饰器
```
def checkParams(name):
    def _deo(fn):
        def wrapper(a,b):
            if isinstance(a,(int,float)) and isinstance(b,(int,float)):
                return fn(a,b)
            print ('variable a and b not be added %s'%name)
            return
        return wrapper
    return _deo
```
其中个关于闭包保存变量到上下文的概念其中滋味还需自己品味. 调用如下
```code
@checkParams('eric')
def add(a,b):
    return a+b
print add
num = add(1,'a')
```
##### 闭包的使用场景
这篇文章引用别处代码较多,也都是一些简答的概念.就不啰嗦了.
参考下:
* 日志打印
* 计时 一个计时的装饰器
```code
def timethis(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(func.__name__, end - start)
        return result
    return wrapper
```
* 缓存装饰器
```code
def cahce(func):
    def wrap(*args):
        name = func.__name__        name = name+''.join(map(str,args[1:]))
        value = redis_client.get(name=name)
        if not value:
            print 'no value from catche %s' % name
            value = func(*args)
            redis_client.set(name=name,value=value,ex=FM_REDIS_TIMEOUT)
        else:
            value = eval(value)
        return value
    return wrap
```

#### 关于上面例子中 @wraps(func)的解释
当然wraps 也是一个装饰器,这个装饰器主要是保留被装饰函数的一些元信息.
这个函数主要做的事情 主要是两个方法
[partial 和 update_wrapper](https://segmentfault.com/a/1190000009398663)

#### 😄
上面都是当前的理解,如果有什么错误,请多多指教,以后有错错漏,我一会及时更新.



