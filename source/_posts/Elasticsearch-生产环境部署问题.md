---
title: Elasticsearch 生产环境部署问题
meta: 'ios,python'
date: 2018-06-26 15:20:42
tags: ElasticSearch
categories: ElasticSearch
keywords: ElasticSearch
---

# 修改监听端口，可以让外部访问
  修改　config/elasticsearch.yml 中的　network.host: 0.0.0.0

  这里会报一个错误：
  ```
  ERROR: [1] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

  ```
[引用](http://kael-aiur.com/elk/ES%E9%85%8D%E7%BD%AE%E7%BB%99%E5%A4%96%E9%83%A8%E6%9C%BA%E5%99%A8%E9%80%9A%E8%BF%87ip%E8%AE%BF%E9%97%AE.html)
[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/vm-max-map-count.html)
```
sudo sysctl -w vm.max_map_count=262144
```

Maximum map count check
从前一点开始，为了有效地使用mmap，Elasticsearch还需要创建许多内存映射区域的能力。最大映射计数检查检查内核允许进程具有至少262,144个内存映射区域，并且仅在Linux上实施。要通过最大映射计数检查，您必须通过sysctl配置vm.max_map_count至少为262144。
[参考](https://my.oschina.net/shibuyisha/blog/983723)


```
ERROR: [1] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
```
修改
```
sudo vi  /etc/security/limits.conf
```
```
#<domain>      <type>  <item>         <value>
#

#*               soft    core            0
#root            hard    core            100000
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#ftp             -       chroot          /ftp
#@student        -       maxlogins       4

algorithm       -       nofile          65536
```

查看 文件描述符数量
```
ulimit -Hn
```


# 用户管理 X-PACK

修改 配置，启用 X-PACK
```
xpack.security.enabled: true
```
[x-park security settings](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/security-settings.html)

```
自动生成密码
bin/elasticsearch-setup-passwords auto
```
[安装X-PACK 步骤](https://www.elastic.co/downloads/x-pack)
```
Changed password for user kibana
PASSWORD kibana = tH5fcQRqKzZv1cV22Bem

Changed password for user logstash_system
PASSWORD logstash_system = 1d9oK8ZMpu1g8rgYfJQz

Changed password for user beats_system
PASSWORD beats_system = ub3aZi57efb8rroI0XFh

Changed password for user elastic
PASSWORD elastic = m7YH8L9eNA7WpFhoD688
```


## 创建一个用于 在logstash 端输入的用户
   1.创建一个role
![roles](http://p1z7ufsgk.bkt.clouddn.com/63e26a4995f4c9eb01fcf7d9d662809c.png)

  manage 和 transport_cliend 具体含义
  [权限roles](https://www.elastic.co/guide/en/shield/2.4/shield-privileges.html#privileges-list-cluster)
  注意给定的索引 以及权限 ，这里给的是logstash* 和 all (读写等权限)
  2.创建 一个用户

  3.logstash 中配置 创建的用户和密码
  ```
  input {
    stdin{}  
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
      	user => "eric"
      	password => "eric123"
    }
   stdout{
        codec => rubydebug
    }
 }
  ```

# 集群节点的 版本要一致
  1.开启 x-pack 以后 集群通讯要认证
  ```
  xpack.security.enabled: true
  xpack.security.audit.enabled: true
  xpack.security.transport.filter.allow: ["10.11.255.120"]

  ```
  2.如果集群之间要自动发现节点需要安全认证
  [ip_filter](https://www.elastic.co/guide/en/x-pack/6.2/ip-filtering.html)


# logstash 使用socket 接受日志信息
从python logging 输出的日志
```
 socket_handle = logging.handlers.SocketHandler(host='10.11.255.120',port=9002)
 socket_formatter = '%(asctime)s [%(levelname)s] [%(filename)s:%(lineno)d] [%(funcName)s]- %(message)s'
   socket_handle.setFormatter(socket_formatter)
 logging.root.addHandler(socket_handle)
```
如下图所示，都是
![socket](http://p1z7ufsgk.bkt.clouddn.com/f8410ee03734c2e139a4b055a6cf9859.png)


如果可以解码 就好了

  1 解决方案，自定写一个logstash 定义一个http 端口，将数据写入到elasticsearch

  ```
  msg = request.form.get('msg','')
  args = request.form.get('args','')
  message = msg % eval(args)
  ```
  我用Flask 框架来做了一个Httpserver这里接受请求，因为使用Python logging通过HttpHandler发送过来的，日志如果使用：
  ```
  log.debug("debug %s %s",'test','haha')
  ```
  这种信息和参数分开来输出日志，接受的参数也是分开的，需要手动处理,具体代码如上面拼接 args实际是一个数组，但是获取的类型是str,需要eval一下

  接受的Form表达打印如下
  ```
  {
    "module": "cclog",
    "relativeCreated": "79.6740055084",
    "process": "17822",
    "threadName": "MainThread",
    "pathname": "/home/yijialiang/project/algorithm_grpc/abcft_algorithm_grpc/cclog.py",
    "thread": "140472267138816",
    "filename": "cclog.py",
    "levelname": "DEBUG",
    "lineno": "105",
    "exc_info": "None",
    "name": "cclog",
    "processName": "MainProcess",
    "msg": "debug %s %s",
    "args": "(u'test', u'haha')",
    "msecs": "419.599056244",
    "funcName": "<module>",
    "exc_text": "None",
    "levelno": "10",
    "created": "1530166912.42"
}
  ```

  可以调用如下代码写入  Elasticsearch
  ```
  from elasticsearch import Elasticsearch
  es = Elasticsearch('http://10.11.255.120:9200',http_auth=('elastic', 'm7YH8L9eNA7WpFhoD688'))
  body = {
    "host":"eric"
  }
  es.index( index='test_index', doc_type="log", body=body )
  ```
