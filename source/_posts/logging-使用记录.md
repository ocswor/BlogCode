---
title: logging 使用记录
meta: ios,python
date: 2018-04-14 09:33:47
tags: logging
categories: Python系列
keywords: logging
---
日志信息很重要，之前一直都是在框架下开发，对于日志这块了解不深。今天总结下。
常用的配置就不说啦。
### 关于Logging的理解
对 logging的工作流程和工作方式这篇文章可以重点看下

[python logging模块使用教程](https://www.jianshu.com/p/feb86c06c4f4)
![图片](https://upload-images.jianshu.io/upload_images/477558-a099cc71d0a4c453.png?imageMogr2/auto-orient/)
>总结：别人总结的我这里直接copy记录一下

* 第一次导入logging模块或使用reload函数重新导入logging模块，logging模块中的代码将被执行，这个过程中将产生logging日志系统的默认配置。

* 自定义配置(可选)。logging标准模块支持三种配置方式: dictConfig，fileConfig，listen。其中，dictConfig是通过一个字典进行配置Logger，Handler，Filter，Formatter；fileConfig则是通过一个文件进行配置；而listen则监听一个网络端口，通过接收网络数据来进行配置。当然，除了以上集体化配置外，也可以直接调用Logger，Handler等对象中的方法在代码中来显式配置。
* 使用logging模块的全局作用域中的getLogger函数来得到一个Logger对象实例(其参数即是一个字符串，表示Logger对象实例的名字，即通过该名字来得到相应的Logger对象实例)。

* 使用Logger对象中的debug，info，error，warn，critical等方法记录日志信息

#### Logger重要模块之一： 日志的处理对象Handdler
```code
logging.StreamHandler: 日志输出到流，可以是sys.stderr、sys.stdout或者文件
logging.FileHandler: 日志输出到文件
日志回滚方式，实际使用时用RotatingFileHandler和TimedRotatingFileHandler
logging.handlers.BaseRotatingHandler
logging.handlers.RotatingFileHandler 达到一定大小之后备份文件。
logging.handlers.TimedRotatingFileHandler  定时备份

logging.handlers.SocketHandler: 远程输出日志到TCP/IP sockets
logging.handlers.DatagramHandler:  远程输出日志到UDP sockets
logging.handlers.SMTPHandler:  远程输出日志到邮件地址
logging.handlers.SysLogHandler: 日志输出到syslog
logging.handlers.NTEventLogHandler: 远程输出日志到Windows NT/2000/XP的事件日志
logging.handlers.MemoryHandler: 日志输出到内存中的制定buffer
logging.handlers.HTTPHandler: 通过"GET"或"POST"远程输出到HTTP服务器
```
#### Logger重要模块之二： Formatter 格式化器
logging.basicConfig的函数各参数：
```code
filename: 指定日志文件名
filemode: 和file函数意义相同，指定日志文件的打开模式，’w’或’a’
format: 指定输出的格式和内容，format可以输出很多有用信息，如上例所示:
%(levelno)s: 打印日志级别的数值
%(levelname)s: 打印日志级别名称
%(pathname)s: 打印当前执行程序的路径，其实就是sys.argv[0]
%(filename)s: 打印当前执行程序名
%(funcName)s: 打印日志的当前函数
%(lineno)d: 打印日志的当前行号
%(asctime)s: 打印日志的时间
%(thread)d: 打印线程ID
%(threadName)s: 打印线程名称
%(process)d: 打印进程ID
%(message)s: 打印日志信息

%(name)s: getLogger(__name__) 记录器的名字

datefmt: 指定时间格式，同time.strftime()
level: 设置日志级别，默认为logging.WARNING
stream: 指定将日志的输出流，可以指定输出到sys.stderr,

sys.stdout或者文件，默认输出到sys.stderr，当stream和filename同时指定时，stream被忽略
```

#### Logger重要模块之三： Filter 过滤器
>Handlers和Loggers可以使用Filters来完成比级别更复杂的过滤。Filter基类只允许特定Logger层次以下的事件。例如用‘A.B’初始化的Filter允许Logger ‘A.B’, ‘A.B.C’, ‘A.B.C.D’, ‘A.B.D’等记录的事件，logger‘A.BB’, ‘B.A.B’ 等就不行。 如果用空字符串来初始化，所有的事件都接受。

![Logger Filer Handler Formatter的关系](https://upload-images.jianshu.io/upload_images/477558-467ca16aae66770f.jpg?imageMogr2/auto-orient/)

#### Logger重要模块之四： Logger 记录器
>Logger是一个树形层级结构，在使用接口debug，info，warn，error，critical之前必须创建Logger实例，即创建一个记录器，如果没有显式的进行创建，则默认创建一个root logger，并应用默认的日志级别(WARN)，处理器Handler(StreamHandler，即将日志信息打印输出在标准输出上)，和格式化器Formatter(默认的格式即为第一个简单使用程序中输出的格式)。

```code
创建方法: logger = logging.getLogger(logger_name)
```
##### 关于 Looger的 树形层级结构
>创建Logger对象。日志记录的工作主要由Logger对象来完成。在调用getLogger时要提供Logger的名称（注：多次使用相同名称 来调用getLogger，返回的是同一个对象的引用。），Logger实例之间有层次关系，这些关系通过Logger名称来体现，如：
>用户使用logging.getLogger([name])获取logger实例。 
如果没有名字，返回logger层级中的根logger（root logger）。以相同名字调用该函数总是返回同一个logger实例。这意味着logger实例不需要在应用的各个部分之间传来传去。
```code
p = logging.getLogger("root")
c1 = logging.getLogger("root.c1")
c2 = logging.getLogger("root.c2")
```
>例子中，p是父logger, c1,c2分别是p的子logger。c1, c2将继承p的设置。如果省略了name参数, getLogger将返回日志对象层次关系中的根Logger。

创建Logger实例后，可以使用以下方法进行日志级别设置，增加处理器Handler。

logger.setLevel(logging.ERROR) # 设置日志级别为ERROR，即只有日志级别大于等于ERROR的日志才会输出
logger.addHandler(handler_name) # 为Logger实例增加一个处理器
logger.removeHandler(handler_name) # 为Logger实例删除一个处理器


