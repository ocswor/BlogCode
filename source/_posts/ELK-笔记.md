---
title: ELK 笔记
meta: ios,python
date: 2018-06-12 16:53:04
tags: ELK
categories: 脚本-测试-工具
keywords: ELK
---

# ELK 练习

[参考文档](https://www.elastic.co/guide/en/kibana/6.x/tutorial-load-dataset.html)


Kibana Console
```
GET _search
{
  "query": {
    "match_all": {}
  }
}

```


supervisord + logstash
```
[program:elkpro_1]
environment=LS_HEAP_SIZE=5000m
directory=/opt/logstash
command=/opt/logstash/bin/logstash -f /etc/logstash/pro1.conf --pluginpath /opt/logstash/plugins/ -w 10 -l /var/log/logstash/pro1.log
[program:elkpro_2]
environment=LS_HEAP_SIZE=5000m
directory=/opt/logstash
command=/opt/logstash/bin/logstash -f /etc/logstash/pro2.conf --pluginpath /opt/logstash/plugins/ -w 10 -l /var/log/logstash/pro2.log
```
logstash 测试启动
```
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```
从控制台输出到控制台


# Elasticsearch

  ### 新增一个文档
  对于雇员目录，我们将做如下操作：

  每个雇员索引一个文档，包含该雇员的所有信息。
  每个文档都将是 employee 类型 。
  该类型位于 索引 megacorp 内。
  该索引保存在我们的 Elasticsearch 集群中。
  ```
  PUT /megacorp/employee/1
  {
      "first_name" : "John",
      "last_name" :  "Smith",
      "age" :        25,
      "about" :      "I love to go rock climbing",
      "interests": [ "sports", "music" ]
  }
  ```
  注意，路径 /megacorp/employee/1 包含了三部分的信息：

  megacorp索引名称    employee类型名称   1特定雇员的ID

  ### 检索文档
  
  ```
  GET /megacorp/employee/1
  ```
