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
支持NSCache 和 Disk 两种缓存.
NSCache 一种类似NSDictornary的keyValue容器.可以限制大小,当里面值被驱逐(超过限制大小会被踢出来)出来的,可以通过代理监听
```code
// Create IO serial queue
_ioQueue = dispatch_queue_create("com.hackemist.SDWebImageCache", DISPATCH_QUEUE_SERIAL);

//同步 串行队列 --->当前线程,串行执行.
dispatch_sync(_ioQueue, ^{
        _fileManager = [NSFileManager new];
    });
```

二、具体保存到硬盘的代码
一个异步队列 autoreleasepool中释放内存变量,内存优化.
```code
dispatch_async(self.ioQueue, ^{
            @autoreleasepool {
                NSData *data = imageData;
                if (!data && image) {
                    // If we do not have any data to detect image format, check whether it contains alpha channel to use PNG or JPEG format
                    SDImageFormat format;
                    if (SDCGImageRefContainsAlpha(image.CGImage)) {
                        format = SDImageFormatPNG;
                    } else {
                        format = SDImageFormatJPEG;
                    }
                    data = [[SDWebImageCodersManager sharedInstance] encodedDataWithImage:image format:format];
                }
                [self _storeImageDataToDisk:data forKey:key];
            }
            
            if (completionBlock) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    completionBlock();
                });
            }
        });
```
从硬盘判断硬盘获取,注意这里**在一个串行队列中.** 
```code
- (BOOL)diskImageDataExistsWithKey:(nullable NSString *)key {
    if (!key) {
        return NO;
    }
    __block BOOL exists = NO;
    dispatch_sync(self.ioQueue, ^{
        exists = [self _diskImageDataExistsWithKey:key];
    });
    
    return exists;
}
```

