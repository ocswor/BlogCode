---
title: Haproxy + supervisor 反向代理
meta: Haproxy,supervisor
date: 2018-06-21 10:06:58
tags: Haproxy,supervisor
categories: 运维
keywords: Haproxy,supervisor
---

#Supervisor 基本操作

### 添加配置文件后，更新新的配置到supervisord
```
supervisorctl update  
```


#Haproxy 配置

[入门简介](https://www.jianshu.com/p/c9f6d55288c0)
[配置详情](http://liaoph.com/haproxy-tutorial/)
[配置参考2](https://www.cnblogs.com/dkblog/archive/2012/03/13/2393321.html)
[实例参考](http://wonderflow.github.io/blog/2014/10/28/haproxye7abafe58fa3e698a0e5b084efbc88cliente5a4b4e4b8adurlhoste4bfaee694b9e5908ee8bdace58f91efbc89/)
```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	stats timeout 30s

defaults #默认参数
    mode tcp  #http模式
    timeout connect 5000ms  #连接server端超时5s
    timeout client 50000ms  #客户端响应超时50s
    timeout server 50000ms  #server端响应超时50s

frontend tcp
    bind *:12420
    default_backend servers  #请求转发至名为"servers"的后端服务

backend servers #后端服务servers
    server server1 127.0.0.1:12421 maxconn 32
    server server2 127.0.0.1:12422 maxconn 32
```

##＃　listen 和　frontend backend
以下写法等价于如下的配置，实现功能一样，但是listen是一对一，直接指定前端和后端，Frontend和Backend 区分前端和后端的好处是比较灵活，可以交叉调用，所以建议用前端和后端交叉写
```
frontend  http
    bind *:80
    default_backend    websrv
backend websrv
    balance    roundrobin
    server     web6e 172.18.50.65:80 check
    server     web7e 172.18.50.75:80 check
```

```
listen http
    bind :80
    balance roundrobin
    server     web6e 172.18.50.65:80 check
    server     web7e 172.18.50.75:80 check
```

```
listen  admin_status           #Frontend和Backend的组合体,监控组的名称，按需自定义名称
        bind 0.0.0.0:8888              #监听端口
        mode http                      #http的7层模式
        log 127.0.0.1 local3 err       #错误日志记录
        stats refresh 5s               #每隔5秒自动刷新监控页面
        stats uri /admin?stats         #监控页面的url访问路径
        stats realm itnihao\ welcome   #监控页面的提示信息
        stats auth admin:admin         #监控页面的用户和密码admin,可以设置多个用户名
        stats auth admin1:admin1       #监控页面的用户和密码admin1
        stats hide-version             #隐藏统计页面上的HAproxy版本信息  
        stats admin if TRUE            #手工启用/禁用,后端服务器(haproxy-1.4.9以后版本)
```

### 获取原始客户端的访问ip信息
```
global
        log /dev/log    local0
        log /dev/log    local1 notice

defaults #默认参数
    mode http  #http模式
    timeout connect 5000ms  #连接server端超时5s
    timeout client 50000ms  #客户端响应超时50s
    timeout server 50000ms  #server端响应超时50s

frontend http-in
    option forwardfor
    bind *:9002
    default_backend servers  #请求转发至名为"servers"的后端服务

backend servers #后端服务servers
    balance roundrobin
    option forwardfor header X-Client
    server server1 127.0.0.1:9004 check maxconn 3000
    server server2 127.0.0.1:9003 check maxconn 3000
```
增加 option forwardfor 字段


```
#check inter 1500是检测心跳频率,rise 3是3次正确认为服务器可用，fall 3是3次失败认为服务器不可用，weight代表权重
 server db2 10.10.10.17:27017 check inter 1500 rise 3 fall 3 weight 2
```
