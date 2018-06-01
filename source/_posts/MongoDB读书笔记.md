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