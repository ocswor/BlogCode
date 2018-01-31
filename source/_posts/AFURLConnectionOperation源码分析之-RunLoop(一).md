---
title: AFURLConnectionOperation源码分析之 RunLoop(一)
meta: ios,python,AFNetowrking2.6.3 AFURLConnectionOperation源码分析(一)
date: 2018-01-30 17:10:28
tags: iOS,RunLoop
categories: iOS系列
keywords: AFURLConnectionOperation源码分析之 RunLoop(一),RunLoop
---

也不知道现在分析AFNetworking 还有人看吗? 😄 网上百度一大堆,怎么才能分析的清新脱俗呢?太难了,不分析也不行,不分析,怎么找工作啊,面试官也不知道你会分析源码,😔 .为了面试,让大家知道我也会分析,我就硬着头皮总结一波,各位随意.

#### 为什么分析2.6.3版本
嘿嘿,这个我就来介绍一波.

其一,因为这个源码比较全,AFHTTPRequestOperationManager 也有 AFHTTPSessionManager,在3.x版本中,AFNetworing移除了AFHTTPRequestOperationManager的相关代码.因为**NSURLConnection**这个用来发送网络请求的方法在9.0以上版本被废除了.不推荐使用了.

其二,主要是因为2.6.3版本的AFHTTPRequestOperationManager 是在2.x系列的最巅峰版.而我这这篇文章里主要是总结下AFURLConnectionOperation在处理异步网络请求使用常驻线程,这一精华部分.

#### RunLoop
嘿嘿,Runloop这个是AFNetworking 在2.x 源码分析系列必须要提的一个G点.

Runloop故名思议,事件循环嘛.我在2015年就总结过一篇,一个线程一个loop循环嘛(你不说2015别人也不知道啊,还以为你现在才听过Runloop呢)
果断将那时候总结的复制一波如下.
>总结要点:
1.每个线程创建的时候,都有一个Runloop循环.
2.每个线程,包括程序的主线程都与之对应的run loop Object.只有辅助线程才需要显式的运行它的run loop.在Cocoa程序中,主线程会主动创建并运行它run loop;
3.Run loop,是一个循环,你的线程进入,并使用它来运行响应输入事件的事件处理.也就是说,代码要提供实现循环的控制语句.换言之就是要有whlie或for循环语句来驱动run loop.在你的循环中,使用run loop object来运行事件处理的代码.它响应接受到得事件并启动已经安装的处理程序.
Run Loop模式
Run Loop 模式是所有要监视的输入源和定时源以及要通知的run loop注册观察者的集合.每次运行你的run Loop,你都要指定(无论显式还是隐式)其运行模式.
在Run Loop运行过程中,只有和模式相关的源才会被监视,并允许他们传递事件消息.
模式可以被指定任意名字,但是模式的内容则不能是任意的.必须添加一个或多个输入源,定时源或者runloop 的观察者到你新建的模式中让他有价值,
注意：模式区分基于事件的源而非事件的种类。例如，你不可以使用模式只选择处理鼠标按下或者键盘事件。你可以使用模式监听端口，暂停定时器或者改变其他源或者当前模式下处于监听状态run loop观察者。
输入源的种类:基于端口的输入源和自定义输入源.
基于端口的输入源监听程序相应的端口。自定义输入源则监听自定义的事件源。
自定义输入源:必须使用Core Foundation里面的CFRunLoopSourceRef类型相关的函数来创建。你可以使用回调函数来配置自定义输入源。Core Fundation会在配置源的不同地方调用回调函数，处理输入事件，在源从run loop移除的时候清理它。

这里还要带两张图,不然别人不知道你会,😄
##### Runloop架构
![Runloop架构](https://images0.cnblogs.com/blog2015/317650/201506/171000560455268.png)
##### Runloop运行时
![Runloop运行时](https://images0.cnblogs.com/blog2015/317650/201506/171000054355761.png)
献丑了,很明显那时候总结的还很青涩,很多事情,文字表达一方面,用代码表达又是另一回事,下面开始show code.不过代码不是我的,是AFNetworing下关于Runloop的使用.

```code
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        //添加端口 监听 保持loop不退出
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}

+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    
    return _networkRequestThread;
}
```
networkRequestThread 方法 一个单例写法,创建一个线程并启动.并且在这里线程上显式的开启了这个runloop循环.
😭 实在不知道这个代码怎么解释了,该解释的已经注释了.
为了解释Runloop 在下翻遍了文档和百度上的资料,得到了一个惊人的结论.runloop 没啥卵用.嘘,请让我把这个B装完.再打我.
没啥卵用是因为我们日常开发90%都不会用到.

#### 关于Runloop 目前我了解到的几个使用场景
-.常驻线程  就是上面AFNetworking的代码 然后啪啪 使用下面的方法对线程进行通讯,让你在这个线程尽情发挥吧 
可以回收一些缓存,变量,进行一些检测,都可以在这个常驻线程上旋转跳跃.
``` code
[self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
```

二.性能优化,在TableView滑动的时候,等待滑动停止再加载图片.这个[git有个demo](https://github.com/ocswor/RunloopDemo)
在线程上开启线程观察
```code
//添加runloop监听者
- (void)addRunloopObserver{
    
    //    获取 当前的Runloop ref - 指针
    CFRunLoopRef current =  CFRunLoopGetCurrent();
    
    //定义一个RunloopObserver
    CFRunLoopObserverRef defaultModeObserver;
    CFRunLoopObserverContext context = {
        0,
        (__bridge void *)(self),
        &CFRetain,
        &CFRelease,
        NULL
    };
    
    //    创建观察者
    //    这里观察的是所有的活动状态
    defaultModeObserver = CFRunLoopObserverCreate(NULL,
                                                  kCFRunLoopAllActivities, YES,
                                                  NSIntegerMax - 999,
                                                  &Callback,
                                                  &context);
    
    //添加当前runloop的观察着
    CFRunLoopAddObserver(current, defaultModeObserver, kCFRunLoopDefaultMode);
    
    //释放
    CFRelease(defaultModeObserver);
}
```

```code
//回调方法
static void Callback(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info){
    
    //通过info桥接为当前的对象
    ERWorkerClass * runloop = (__bridge ERWorkerClass *)info;
    NSString *activityDescription;
    switch (activity) {
        case kCFRunLoopEntry:
            activityDescription = @"kCFRunLoopEntry";
            break;
        case kCFRunLoopBeforeTimers:
            activityDescription = @"kCFRunLoopBeforeTimers";
            break;
        case kCFRunLoopBeforeSources:
            activityDescription = @"kCFRunLoopBeforeSources";
            break;
        case kCFRunLoopBeforeWaiting:
            activityDescription = @"kCFRunLoopBeforeWaiting";
            //可以在 waiitng 之前处理一些任务
            break;
        case kCFRunLoopAfterWaiting:
            activityDescription = @"kCFRunLoopAfterWaiting";
            break;
        case kCFRunLoopExit:
            activityDescription = @"kCFRunLoopExit";
            break;
        default:
            break;
    }
    NSLog(@"%@",activityDescription);
 
}
```
三.这个场景 应该是在网络通讯上,轮询任务 没有具体实战,具体就不描述了,暂且属于保留项目.

四.提到Runloop 不提一口NSPort,有点说不过去. 在iOS上只能使用NSMachPort  即使你 myPort = [NSPort port]; 也是返回一个NSMachPort.我听说的.
NSPort 就是用来进行线程通讯的.port ---> remotePort 通讯嘛
```code
- (void)setup{
    NSLog(@"%@",[[NSRunLoop currentRunLoop] currentMode]);
    //创建并启动一个 thread
    self.thread = [[NSThread alloc]initWithTarget:self selector:@selector(threadTest) object:nil];
    [self.thread setName:@"Test Thread"];
    [self.thread start];
    
    //向 RunLoop 发送消息的简便方法，系统会将消息传递到指定的 SEL 里面
//    [self performSelector:@selector(receiveMsg) onThread:self.thread withObject:nil waitUntilDone:NO];
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        //直接去的子线程 RunLoop 的一个 port，并向其发送消息，这个是比较底层的 NSPort 方式进行线程通信
//        [self.port sendBeforeDate:[NSDate date] components:[@[[@"hello" dataUsingEncoding:NSUTF8StringEncoding]] mutableCopy] from:_mainPort reserved:12345];
        [self.port sendBeforeDate:[NSDate date] msgid:101 components:[@[[@"hello" dataUsingEncoding:NSUTF8StringEncoding]] mutableCopy] from:_mainPort reserved:0];
    });
}
```

```code
- (void)threadTest{
    NSLog(@"%@",@"child thread start");
    //threadTest 这个方法是在 Test Thread 这个线程里面运行的。
    NSLog(@"%@",[NSThread currentThread]);
    //获取这个线程的 RunLoop 并让他运行起来
    NSRunLoop* runloop = [NSRunLoop currentRunLoop];
    self.port = [[NSPort alloc]init];
    self.port.delegate = self;
    [runloop addPort:self.port forMode:NSRunLoopCommonModes];
    //约等于runUntilDate:[NSDate distantFuture]
    [runloop run];
}

```

接受消息的代理实现
```code
#pragma NSMachPortDelegate
- (void)handleMachMessage:(void *)msg{
    NSLog(@"handle message thread:%@",[NSThread currentThread]);
    NSLog(@"message:%d", *(int *)msg);
//    NSLog(@"message:%@", (__bridge NSString *)msg);
  
}
```

总结一波,还是看代码吧,git

参考链接:
[苹果官方文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW7)
[别人家的分析](http://www.cnblogs.com/zy1987/p/4582466.html)
