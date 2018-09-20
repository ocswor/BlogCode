---
title: Python httplib
meta: Python
date: 2018-07-06 16:52:44
tags: Python
categories: Python系列
keywords: httplib 长连接
---

# 长连接代码实现

```
import httplib
headers = {"Content-type": "application/x-www-form-urlencoded","Accept": "text/plain","Connection":"Keep-Alive"}
conn = httplib.HTTPConnection("192.168.54.138",6040)
for i in range(self.times):
   conn.request("POST", "/index.html", self.data, headers)
   response = conn.getresponse()
   response.read()
conn.close()
```

主要在于 headers 声明 Connection:Keep-Alive
每次请求 不用再连续创建 HTTPConnection


#
