---
title: Linux 下的七七八八
meta: Linux
date: 2018-06-14 20:54:42
tags: Linux
categories: Linux
keywords: Linux
---

### 文件权限

文件权限数字说明：
 -（代表类型）×××（所有者）×××（组用户）×××（其他用户）

常用的设置权限
```
sudo chmod 600 ××× （只有所有者有读和写的权限）

sudo chmod 644 ××× （所有者有读和写的权限，组用户只有读的权限）

sudo chmod 700 ××× （只有所有者有读和写以及执行的权限）

sudo chmod 666 ××× （每个人都有读和写的权限）

sudo chmod 777 ××× （每个人都有读和写以及执行的权限）
```
### 性能分析

vmstat是一个查看虚拟内存（Virtual Memory）使用状况的工具，使用vmstat命令可以得到关于进程、内存、内存分页、堵塞IO、traps及CPU活动的信息。


### linux 编译 Haproxy

```
 wget https://github.com/haproxy-unofficial-obsolete-mirrors/haproxy/archive/v1.7.0.tar.gz
tar -zxvf v1.7.0.tar.gz
sudo apt-get install gcc  autoconf automake -y
sudo apt-get install make
make TARGET=linux2628 arch=x86_64 USE_LINUX_TPROXY=1 PREFIX=/opt/algorithm/haproxy
make install PREFIX=/opt/algorithm/haproxy

make clean 删除 make文件
```
