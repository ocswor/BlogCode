---
title: iOS 蓝牙接口说明
meta: ios,python
date: 2018-03-12 17:12:52
tags: iOS蓝牙接口调用
categories: iOS系列
keywords: iOS蓝牙接口说明
---
蓝牙 一个想说又说不出口的东西,摸着自己的胸,深吸一口气.

### 楔子
在iOS上使用蓝牙开发,现在比较出名,文档比较完善的就是**BabyBluetooth** 这个轮子库.不了解的可以在gitHub上搜一下就看到了.但是,只看BabyBluetooth的文档,各种链式调用,block回调,如果不懂官方的CBCentralManager,CBPeripheral,Seivices,Characteristics的关系也是云里雾里的.这么好的工具,感觉用起来很不顺手,总是感觉心里不踏实.

所以这篇文章重要介绍下官方CoreBluetooth 使用的各个接口,后面有一个自己封装的一个demo,就算不使用BabyBluetooth也一样可以自己封装一个自己想要的蓝牙通讯工具库.
![关系图](http://p1z7ufsgk.bkt.clouddn.com/%E6%9C%AA%E5%91%BD%E5%90%8D.001.jpeg)
### 一 、初始化

与其说BCentralManager ,CBPeripheral,Seivices,Characteristics 的一些关系不如说我们开发蓝牙需要准备的一些前提条件.

首先我要来到从梦开始的地方,第一步初始化一个 CBCentralManager
这个可以理解是把手机看做一个蓝牙manager,实现初始化
```code
_blueToothManager = [[CBCentralManager alloc]initWithDelegate:self queue:nil];
```
### 二 、开始扫描
上面👆初始化之后,就直接回调,当前蓝牙的状态,打开,关闭等
```code
-(void)centralManagerDidUpdateState:(CBCentralManager *)central
{
    switch (central.state) {
        case CBCentralManagerStatePoweredOn:
            if (_scanTimeout>0) {
                _scanTimer = [NSTimer scheduledTimerWithTimeInterval:_scanTimeout target:self selector:@selector(scanTimeoutHandler:) userInfo:nil repeats:NO];
            }
        {
            NSArray *nServices = [NSArray arrayWithObjects:[CBUUID UUIDWithString:kServiceUUID], nil];
            NSDictionary *nOptions = [NSDictionary dictionaryWithObject:[NSNumber numberWithBool:YES] forKey:CBCentralManagerScanOptionAllowDuplicatesKey];
            //开始扫描周围设备
            [_blueToothManager scanForPeripheralsWithServices:nil options:nOptions];

        }
           
            break;
        default:
            // 断开连接后，需要提醒断开
            LOG(@"CBManagerStatePoweredOff");
         
            break;
    }
}
```

### 三、连接和查找service

```code
// 发现蓝牙外设。。
-(void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary *)advertisementData RSSI:(NSNumber *)RSSI
{
    LOG(@"%@ uuid = %@",peripheral.name,peripheral.identifier);
    if ([peripheral.name isEqualToString:@"ble_ai_toy"]) {
        [_blueToothManager connectPeripheral:peripheral options:nil];
        [_blueToothManager stopScan];
        LOG(@"开始连接");
         _peripheral = peripheral;
    }
    
}

// 未能成功连接蓝牙
-(void)centralManager:(CBCentralManager *)central didFailToConnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error
{
    LOG(@"didFailToConnectPeripheral");
}

// 连接上蓝牙。。
-(void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral
{
 
   
    [peripheral setDelegate:self];
    NSArray *services = @[[CBUUID UUIDWithString:kServiceUUID]];
//    NSArray *services = nil;
    [peripheral discoverServices:services];
    LOG(@"连接成功");
    
}
-(void)centralManager:(CBCentralManager *)central didDisconnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error{
    LOG(@"断开连接");
}
```
以上就是 CBCentralManagerDelegate 的相关方法.手机作为主动连接方manager,找到了我们需要连接的其他蓝牙外设.所以在上面我们需要使用一个全局变量来保存这个外设.以备后面查找特征值使用
```code
 _peripheral = peripheral;保存引用 在上面👆正式连接成功的回调方法中实现.
```
### 四、发现外设的相关service的同时,并查找Characteristic,并在发现特征值下,开始监听该特征值的变化.具体代码如下👇

```code
//查找 Services
-(void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error
{
    
    if (error) {
        LOG(@"Error discovering service:%@", [error localizedDescription]);
        return;
    }
     LOG(@"发现 相关 services ");
    for (CBService *service in peripheral.services) {
        if ([service.UUID isEqual:[CBUUID UUIDWithString:kServiceUUID]]) {
            [peripheral discoverCharacteristics:@[[CBUUID UUIDWithString:kCharacteristic1UUID],[CBUUID UUIDWithString:kCharacteristic2UUID]] forService:service];
        }
       
    }
}
//查找 Services Characteristics
-(void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error
{
    if (error) {
        LOG(@"Error discovering characteristics:%@", [error localizedDescription]);
        return;
    }
   LOG(@"发现 相关 特征 ");
    for (CBService *service in peripheral.services) {
        if ([service.UUID isEqual:[CBUUID UUIDWithString:kServiceUUID]]) {
            for(CBCharacteristic *characteristic in service.characteristics)
            {
                if ([characteristic.UUID isEqual:[CBUUID UUIDWithString:kCharacteristic2UUID]]) {
                    [peripheral setNotifyValue:YES forCharacteristic:characteristic];
                    LOG(@"开始监听2 特征值");
                }
                if ([characteristic.UUID isEqual:[CBUUID UUIDWithString:kCharacteristic1UUID]]) {
                    [peripheral setNotifyValue:YES forCharacteristic:characteristic];
                    LOG(@"开始监听1 特征值");
                }
            }
        }
        
    }
}
```
### 五、读取特征值 和 获取特征值
```code
//更新通知
-(void)peripheral:(CBPeripheral *)peripheral didUpdateNotificationStateForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error
{
    LOG(@"didUpdateNotification");
    if ([characteristic.UUID isEqual:[CBUUID UUIDWithString:kCharacteristic1UUID]]) {
        [peripheral readValueForCharacteristic:characteristic];
    }
    if ([characteristic.UUID isEqual:[CBUUID UUIDWithString:kCharacteristic2UUID]]) {
        [peripheral readValueForCharacteristic:characteristic];
    }
}
//更新value
-(void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{
    if (error) {
        LOG(@" Error in didUpdateValueForCharacteristic=%@",error.description);
        return;
    }
    
    LOG(@"特征值 更新");
}

```


