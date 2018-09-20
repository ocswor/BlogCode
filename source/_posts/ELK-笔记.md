---
title: ELK 笔记（基础入门）
meta: ios,python
date: 2018-06-12 16:53:04
tags: ELK,elasticsearch
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

修改日志级别
```
PUT /_cluster/settings
{
    "transient" : {
        "logger.discovery" : "DEBUG"
    }
}
```

### 实战配置 本机验证通过的logstash配置文件
```
input {
    file {
    	path => "/var/algorithm/logs/*.logs"
   	start_position => "beginning"
    }  
}

filter{
    #mutate{add_field => {"[@metadata][debug]"=>true}}

    grok{
        match => {
            "message" => '%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:[@metadata][timestamp]}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})" %{NUMBER:response} (?:%{NUMBER:bytes}|-) %{QS:referrer} %{QS:agent}'
        }
    }
}

output {

   elasticsearch {
        hosts => ["127.0.0.1"]
        manage_template => true
        template_overwrite => true
    }
   stdout{
        codec => rubydebug
    }
 }
```
可以通过以下命令查看 具体数据
```
GET logstash-2018.06.13/_search
```

日志信息是一个临时文件，一次测试完毕，以后，可以通过复制添加数据，模拟新增日志的输出。

可以通过以下命令查看索引的状态以及数量：
```
http://localhost:9200/_cat/indices?v
```

查看当前模板
```
 curl -XGET 'http://127.0.0.1:9200/_template/logstash'
```
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
  ### 轻量搜素
  ```  
  GET /megacorp/employee/_search
  ```
  ```
  GET /megacorp/employee/_search?q=last_name:Smith
  ```
  ### 使用表达式搜索
  ```
  GET /megacorp/employee/_search
  {
      "query" : {
          "match" : {
              "last_name" : "Smith"
          }
      }
  }
  ```
  ### 更复杂的搜索
  ```
  GET /megacorp/employee/_search
  {
      "query" : {
          "bool": {
              "must": {
                  "match" : {
                      "last_name" : "smith"
                  }
              },
              "filter": {
                  "range" : {
                      "age" : { "gt" : 30 }
                  }
              }
          }
      }
  }
  ```
  这部分是一个 range 过滤器 ， 它能找到年龄大于 30 的文档，其中 gt 表示_大于(great than)。

  ### 全文搜索
  传统数据库确实很难搞定的任务。
  ```
  GET /megacorp/employee/_search
  {
      "query" : {
          "match" : {
              "about" : "rock climbing"
          }
      }
  }
  ```

[关于相关性和得分结果排序](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_full_text_search.html#_full_text_search)

 ### 短语搜索
 要精确匹配一系列单词或者短语
 ```
 GET /megacorp/employee/_search
 {
      "query" : {
          "match_phrase" : {
              "about" : "rock climbing"
          }
      }
  }
 ```
  ### 高亮搜索
  ```
  GET /megacorp/employee/_search
  {
      "query" : {
          "match_phrase" : {
              "about" : "rock climbing"
          }
      },
      "highlight": {
          "fields" : {
              "about" : {}
          }
      }
  }
  ```
  当执行该查询时，返回结果与之前一样，与此同时结果中还多了一个叫做 highlight 的部分。这个部分包含了 about 属性匹配的文本片段，并以 HTML 标签 <em></em> 封装：
  ```
  {
     "hits": {
        "total":      1,
        "max_score":  0.23013961,
        "hits": [
           {
              ...
              "_score":         0.23013961,
              "_source": {
                 "first_name":  "John",
                 "last_name":   "Smith",
                 "age":         25,
                 "about":       "I love to go rock climbing",
                 "interests": [ "sports", "music" ]
              },
              "highlight": {
                 "about": [
                    "I love to go <em>rock</em> <em>climbing</em>"
                 ]
              }
           }
        ]
     }
  }
  ```

多个字段组成的查询
  ```
  GET pdfextract-2018.08.06/_search
{
  "query": {
    "match": {
      "level": "INFO"
    }
  }
}

GET pdfextract-2018.08.06/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "level": "INFO"
          }
        },
        {
          "match": {
            "host": "alg-service117"
          }
        }
      ]
    }
  }
}
  ```
  [关于高亮的详细定制参考](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/search-request-highlighting.html)
