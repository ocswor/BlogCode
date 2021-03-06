---
title: HTTP详解笔记
meta:  TCPIP
date: 2018-05-24 14:57:08
tags: TCP/IP
categories: 网络通讯
keywords: TCP/IP
---

# 网络基础
协议(protocol) 在计算机和网络设备直接互相通讯，一切的基础都归根于协议，从探测，到通讯目标，谁先发？怎么发，怎么结束？　有需要有一种规则。无规矩不成方圆，协议就应运而生了。

网络设施的构建，一切都要用规矩说话，所以在不同的层次，不同的结构，不同的设备，存在这不同的协议。

![协议总览](http://p1z7ufsgk.bkt.clouddn.com/d031595a86cb1318650e3163255327aa.png)

>TCP/IP 是指 TCP 和 IP这两种协议，是互联网相关的各类协议族的总称
# HTTP相关的协议
### TCP/IP的分层
   在TCP/IP协议族按层次分为以下４层
   * 应用层　这一层的协议有　
   FTP(File Transfer Protocol,文件传输协议)　和 DNS(Domain Name System,域名系统)　HTTP 协议

   * 传输层　在传输层有两个性质不同的协议:TCP(Transmission Control Protocol,传输控制协议)和 UDP(User Data Protocol,用户数据报协议)。

   * 网络层(又名网络互连层)　　IP 协议
   数据包是网络传输的最小数据单位。该层规定了通过怎样的路径(所谓的传输路线)到达对方计算机,并把数据包传送给对方。IP 协议 增加作为通信目的地的MAC 地址后转发给链路层。这样一来,发往网络的通信请求就准备齐全了。
   * 数据链路层
   用来处理连接网络的硬件部分。包括控制操作系统、硬件的设备驱动、NIC(Network InterfaceCard,网络适配器,即网卡),及光纤等物理可见部分(还包括连接器等一切传输媒介)。硬件上的范畴均在链路层的作用范围之内。

![分层](http://p1z7ufsgk.bkt.clouddn.com/e215ce47872aeeb13bae5c690b0f0967.png)
![协议分层](http://p1z7ufsgk.bkt.clouddn.com/ea44d44559dcb9d3de836d0d2a24a0ed.png)


### 负责传输的IP协议
   >TCP/IP 协议族中的IP 指的就是网际协议,协议名称中占据了一半位置,其重要性可见一斑。

   IP 协议的作用是把各种数据包传送给对方。而要保证确实传送到对方那里,则需要满足各类条件。其中两个重要的条件是 IP 地址和 MAC 地址(Media Access Control Address)。

   IP 地址指明了节点被分配到的地址,MAC 地址是指网卡所属的固定地址。IP 地址可以和 MAC地址进行配对。IP 地址可变换,但 MAC 地址基本上不会更改。

   >ARP 协议(Address ResolutionProtocol)。

   ARP 是一种用以解析地址的协议,根据通信方的 IP 地址就可以反查出对应的 MAC
地址。
![ARP协议](http://p1z7ufsgk.bkt.clouddn.com/8ae70af2b864aef366dafbe30158407a.png)
