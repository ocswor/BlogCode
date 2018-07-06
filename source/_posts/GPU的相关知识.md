---
title: GPU的相关知识
meta: 'ios,python'
date: 2018-06-14 14:50:10
tags: GPU
categories: 脚本-测试-工具
keywords: GPU
---

### 什么是GPU
图形处理器（英语：Graphics Processing Unit，缩写：GPU），又称显示核心、视觉处理器、显示芯片，是一种专门在个人电脑、工作站、游戏机和一些移动设备（如平板电脑、智能手机等）上图像运算工作的微处理器。
[百科链接](https://baike.baidu.com/item/%E5%9B%BE%E5%BD%A2%E5%A4%84%E7%90%86%E5%99%A8/8694767?fromtitle=gpu&fromid=105524)
### 查看GPU的相关信息

一台机器可以安装多张显卡
我们公司安装的是NVIDIA厂商的
可以通过 以下命令查看所有可用的显卡
```
nvidia-smi -L
```

Linux查看显卡信息：
```
lspci | grep -i vga
```
使用nvidia GPU可以：
```
lspci | grep -i nvidia
```
