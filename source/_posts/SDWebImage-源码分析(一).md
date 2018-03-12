---
title: SDWebImage 源码分析(一)
meta: ios,python
date: 2018-03-08 16:16:09
tags:
categories:
keywords:
---
#### 先分析 Cache
##### 主要是SDImageCache,SDImageCacheConfig 这两个类
将cache 的config配置独立出来.

一、构造接口
```code
+ (nonnull instancetype)sharedImageCache;
- (nonnull instancetype)initWithNamespace:(nonnull NSString *)ns;
- (nonnull instancetype)initWithNamespace:(nonnull NSString *)ns
                       diskCacheDirectory:(nonnull NSString *)directory
```

在构造方法时候初始化一个serial queue
```code
// Create IO serial queue
_ioQueue = dispatch_queue_create("com.hackemist.SDWebImageCache", DISPATCH_QUEUE_SERIAL);

//同步 串行队列
dispatch_sync(_ioQueue, ^{
        _fileManager = [NSFileManager new];
    });
```


