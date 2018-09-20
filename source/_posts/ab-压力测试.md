---
title: ab 压力测试
meta: ab
date: 2018-07-06 20:15:55
tags: ab
categories: 脚本-测试-工具
keywords: ab
---

# 相关参数说明
```
-n requests     Number of requests to perform
-c concurrency  Number of multiple requests to make at a time
```
而且：
Cannot use concurrency level greater than total number of requests

```
ab -c 2 -n20 http://127.0.0.1:9001/test
```
总共发出20个请求，并发２,模拟两个客户端




# 返回结果

```
Requests per second:    2711.45 [#/sec] (mean)
```
吞吐率　一秒处理多少请求


```
Time per request: 328.016 [ms] (mean) //第一个
Time per request: 109.339 [ms] (mean, across all concurrent requests) //第二个
```
其实主要的区别在请求的时间上，第一个Time per request会计算每一个请求的请求和结束时间，注意区别在计算统计每一个请求。
举个例子：
我们并发请求3个，分别每个请求的时间为30ms、40ms 、50ms ，总共并发完成的时间为90ms,那么：
(第二个）Time per request ＝ 90ms /3 = 30ms
(第一个) Time per request = (30ms + 40ms + 50ms) /3 = 40ms

```
/*大家最关心的指标之一，指的是吞吐率
相当于 LR 中的 每秒事务数 ，后面括号中的 mean 表示这是一个平均值*/  
Requests per second:    13.45 [#/sec] (mean)
/*大家最关心的指标之二，指的是用户平均请求等待时间
相当于 LR 中的 平均事务响应时间 ，后面括号中的 mean 表示这是一个平均值*/
Time per request:       743.726 [ms] (mean)
/*大家最关心的指标之三，指的是服务器平均请求处理时间
Time per request:       74.373 [ms] (mean, across all concurrent requests)  
```
# Post 请求接口
```
ab -c 200 -n 1000 -T 'application/x-www-form-urlencoded' -p postdata.txt http://domain/test.php

ab -c 200 -n 1000 -T 'application/json' -p postdata.txt http://10.11.255.120:9002
```

postdata.txt
```
{"relativeCreated": 404.8421382904053, "process": 9050, "module": "cclog", "funcName": "<module>", "exc_text": null, "name": "cclog", "thread": 139861490800384, "created": 1531101954.971471, "threadName": "MainThread", "msecs": 971.4710712432861, "filename": "cclog.py", "levelno": 40, "processName": "MainProcess", "pathname": "/home/yijialiang/project/algorithm_grpc/abcft_algorithm_grpc/cclog.py", "lineno": 146, "msg": "test", "exc_info": null, "levelname": "ERROR"}
```


访问　Haproxy
```
Time taken for tests:   3.491 seconds
Requests per second:    286.43 [#/sec] (mean)
Time per request:       698.258 [ms] (mean)
Time per request:       3.491 [ms] (mean, across all concurrent requests)
```

访问单个节点
```
Time taken for tests:   2.068 seconds
Requests per second:    483.51 [#/sec] (mean)
Time per request:       413.638 [ms] (mean)
Time per request:       2.068 [ms] (mean, across all concurrent requests)
```

```

Requests per second:    541.27 [#/sec] (mean)
Time per request:       369.503 [ms] (mean)
Time per request:       1.848 [ms] (mean, across all concurrent requests)
```


访问量增大到10000 Haproxy
```
Time taken for tests:   12.164 seconds
Complete requests:      10000
Failed requests:        2056
Requests per second:    822.10 [#/sec] (mean)
Time per request:       243.278 [ms] (mean)
Time per request:       1.216 [ms] (mean, across all concurrent requests)
```

```
Time taken for tests:   7.898 seconds
Complete requests:      10000
Failed requests:        4996
Requests per second:    1266.19 [#/sec] (mean)
Time per request:       157.955 [ms] (mean)
Time per request:       0.790 [ms] (mean, across all concurrent requests)
```
