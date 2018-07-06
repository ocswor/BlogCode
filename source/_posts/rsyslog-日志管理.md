---
title: rsyslog 日志管理
meta: rsyslog
date: 2018-06-22 16:00:22
tags: rsyslog
categories: Linux
keywords: rsyslog
---

# 简介

Rsyslog可以简单的理解为syslog的超集，在老版本的Linux系统中，Red Hat Enterprise Linux 3/4/5默认是使用的syslog作为系统的日志工具，从RHEL 6 开始系统默认使用了Rsyslog。

Rsyslog 是负责收集 syslog 的程序，可以用来取代 syslogd 或 syslog-ng。 在这些 syslog 处理程序中，个人认为 rsyslog 是功能最为强大的。 其特性包括：

支持输出日志到各种数据库，如 MySQL，PostgreSQL，MongoDB，ElasticSearch，等等；
通过 RELP + TCP 实现数据的可靠传输（基于此结合丰富的过滤条件可以建立一种 可靠的数据传输通道供其他应用来使用）；
精细的输出格式控制以及对消息的强大 过滤能力；
高精度时间戳；队列操作（内存，磁盘以及混合模式等）； 支持数据的加密和压缩传输等。
