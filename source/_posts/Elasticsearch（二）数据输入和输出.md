---
title: Elasticsearch（二）数据输入和输出
meta: Elasticsearch
date: 2018-06-13 16:06:06
tags: elasticsearch
categories: 脚本-测试-工具
keywords: elasticsearch
---

# 概要
### 重点笔记

* Elastcisearch 是分布式的 文档 存储。它能存储和检索复杂的数据结构--序列化成为JSON文档--以 实时 的方式。 换句话说，一旦一个文档被存储在 Elasticsearch 中，它就是可以被集群中的任意节点检索到。

* 在 Elasticsearch 中， 每个字段的所有数据 都是 默认被索引的 。 即每个字段都有为了快速检索设置的专用倒排索引。

* 什么是文档? 在 Elasticsearch 中，术语 文档 有着特定的含义。它是指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID。

* 字段的名字可以是任何合法的字符串，但不可以包含时间段。

### 文档元数据
一个文档不仅仅包含它的数据 ，也包含 元数据 —— 有关 文档的信息。 三个必须的元数据元素如下：

* _index文档在哪存放

一个 索引 应该是因共同的特性被分组到一起的文档集合。 例如，你可能存储所有的产品在索引 products 中，而存储所有销售的交易到索引 sales 中。 虽然也允许存储不相关的数据到一个索引中，但这通常看作是一个反模式的做法。
* _type文档表示的对象类别

数据可能在索引中只是松散的组合在一起，但是通常明确定义一些数据中的子分区是很有用的。
*Elasticsearch 公开了一个称为 types （类型）的特性，它允许您在索引中对数据进行逻辑分区。*
可以理解是索引下的一个子分区 对索引的文档的二次区分。

* _id 文档唯一标识

ID 是一个字符串， 当它和 _index 以及 _type 组合就可以唯一确定 Elasticsearch 中的一个文档。 当你创建一个新的文档，要么提供自己的 _id ，要么让 Elasticsearch 帮你生成。_

### 自定义文档

  ```
  PUT /website/blog/123
  {
    "title": "My first blog entry",
    "text":  "Just trying this out...",
    "date":  "2014/01/01"
  }
  ```
  指定**索引文档**的id以上返回结果：
  ```
  {
   "_index":    "website",
   "_type":     "blog",
   "_id":       "123",
   "_version":  1,
   "created":   true
  }
  ```

  自动创建id,使用POST
  ```
  POST /website/blog/
  {
    "title": "My second blog entry",
    "text":  "Still trying this out...",
    "date":  "2014/01/01"
  }
  ```
返回结果如下：
```
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "AVFgSgVHUP18jI2wRx0w",
   "_version":  1,
   "created":   true
}
```
>自动生成的 ID 是 URL-safe、 基于 Base64 编码且长度为20个字符的 GUID 字符串。 这些 GUID 字符串由可修改的 FlakeID 模式生成，这种模式允许多个节点并行生成唯一 ID ，且互相之间的冲突概率几乎为零。

### 取回一个文档
```
GET /website/blog/123?pretty
```
或者
```
curl -i -XGET http://localhost:9200/website/blog/124?pretty
```
### 返回文档的一部分
```
GET /website/blog/123?_source=title,text
```
结果如下：
```
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```
**Note:** source 是 返回文档的内容。这里只有两个字段返回。

### 检查一个文档是否存在
如果只想检查一个文档是否存在 --根本不想关心内容--那么用 HEAD 方法来代替 GET 方法。 HEAD 请求没有返回体，只返回一个 HTTP 请求报头：
```
curl -i -XHEAD http://localhost:9200/website/blog/123
```

### 更新整个文档
在 Elasticsearch 中文档是 不可改变 的，不能修改它们。 相反，如果想要更新现有的文档，需要 重建索引 或者进行替换， 我们可以使用相同的 index API 进行实现

如果创建同一个id的文档会告诉你创建失败：
```
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 2,
  "created":   false
}
```
created :false

即使使用update Api 更改一个文档 也是按照如下流程：

1、从旧文档构建 JSON
2、更改该 JSON
3、删除旧文档
4、索引一个新文档

### 创建新文档
而不是 在创建的时候，覆盖之前的文档 ？
需要显示声明：
```
PUT /website/blog/123?op_type=create
{ ... }
```
或者
```
PUT /website/blog/123/_create
{ ... }
```

### 删除一个文档
```
DELETE /website/blog/123
```

### version 字段
>即使文档不存在（ Found 是 false ）， version 值仍然会增加。这是 Elasticsearch 内部记录本的一部分，用来确保这些改变在跨多节点时以正确的顺序执行。
