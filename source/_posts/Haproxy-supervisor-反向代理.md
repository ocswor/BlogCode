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
