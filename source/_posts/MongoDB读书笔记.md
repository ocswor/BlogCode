---
title: MongoDB读书笔记(一)
meta: MongoDB
date: 2018-06-01 15:15:24
tags: MongoDB
categories: 数据库
keywords: MongoDB
---

# MongoDB 简介

独特功能概览：

   * 索引  
   MongoDB支持通用二级索引,允许多种快速查询,且提供唯一索引、复合索引、地理空间索引,以及全文索引
   * 特殊的集合类型
   MongoDB支持存在时间有限的集合,适用于那些将在某个时刻过期的数据,如会话(session)。类似地,MongoDB也支持固定大小的集合,用于保存近期数据,如日志。


# MongoDB 的安装
[文档链接](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)

修改支持安全认证：
```
sudo vi /etc/mongodb.conf
```
```
security:
  authorization: enabled
```
### 创建用户
```
db.createUser(
  {
    user: "algorithm",
    pwd: "cH7jufath5_a",
    roles: [ { role: "readWrite", db: "cr_data" }]
  }
)
```
```
db.createUser(
  {
    user: "adminUser",
    pwd: "adminPass",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```
### Mongo shell 使用说明
[Management and Authentication API](https://docs.mongodb.com/manual/reference/security/)


# 查缺补漏
  ### 操作符 $or

  $or是一个逻辑or操作符操作在一个数据或者多个表达式并且需要选择至少一个满足条件的表达式，$or有至少以下表达式
  ```
  { $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }
  ```

  下面的例子：
  ```
  db.inventory.find( { $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] } )
  ```
  上面的例子会查询集合inventory中所有字段quantity小于20或者price等于10的所有文档。

  >Note:
  使用$or条件评估条款，MongoDB会扫描整个文档集合，如果所有的条件支持索引，MongoDB进行索引扫描，因此MongoDB使用索引执行$or表达式，$or中的所有表达式必须支持索引，否则的话MongoDB就会扫描整个集合。

  当使用$or查询并且使用索引时，每个$or的条件表达式都可以使用自己的索引，考虑下面的查询：
  ```
  db.inventory.find( { $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] } )
  ```
  支持上面的查询你不需要创建一个符合索引，而是在字段quantity上创建一个索引，在price上创建一个索引。


  项目中使用：
  ```
  rr = coll.find(
            {
                "fileId": {"$lt": last_updated},
                "$or": [
                    {"chartType": "BITMAP_CHART"},
                    {"is_bitmap": True},
                ],
            },
            {
                "_id": True,
                "fileId": True,
                "bitmap_ver": True,
                "state": True,
            },
            sort=[("fileId", -1)],
            limit=n
        )
  ```
  这个则表示 filedId 为必查字段，chartType和 is_bitmap 满足其中之一即可。
![mongo query](http://p1z7ufsgk.bkt.clouddn.com/18d781803074fafe805fe7edb90ad9b6.png)
使用上面条件查询 2400万的数据 用了 0.034秒，可能和这个数据的位置有关，因为查询的filId排名比较靠前

 ### 操作符  $and
 语法：{ $and: [ { <expression1> }, { <expression2> } , ... , {<expressionN> } ] }

$and执行一个逻辑and操作在一个或者多个表达式上，并且查询数组中指定的所有表达式指定的文档document,$and使用短路求值，如果第一个表达式的结果是false，MongoDB将不会执行剩余的表达式；

例如：and查询指定同一个字段的多个查询条件
```
db.inventory.find( { $and: [ { price: { $ne: 1.99 } }, { price: { $exists: true } } ] } )
```
这个查询会选择集合inventory中的所有文档，条件是price不等于1.99并且price字段存在；
以上查询还可以使用隐式AND操作，如下：
```
db.inventory.find( { price: { $ne: 1.99, $exists: true } } )
```
以下字段将会查询price字段值等于0.99或1.99并且sale字段值为true或者qty小于20的所有文档；
使用隐式AND操作无法构建此查询，因为它不止一次使用$or操作；
```
db.inventory.find( {
    $and : [
        { $or : [ { price : 0.99 }, { price : 1.99 } ] },
        { $or : [ { sale : true }, { qty : { $lt : 20 } } ] }
    ]
} )
```


  ### explain
  可以使用explain() 函数查看MongoDB在执行查询的过程中所做的事情。

  ### 线上数据库创建索引 在后台创建，避免锁表
  ```
  db.t1.createIndex({idCardNum:1},{background:1})
  ```
  (线上数据库使用规范)[http://blog.51cto.com/hcymysql/2061451]



  #map_reduce 使用
  ```
map_func = Code("function () {this.tags.forEach(function(z) {emit(z, 1);});}")

reduce_func = Code("function (key, values) {"
                    "  var total = 0;"
                    "  for (var i = 0; i < values.length; i++) {"
                    "    total += values[i];"
                    "  }"
                    "  return total;"
                    "}")
  ```
