---
title: SHELL 常用命令
meta: ios,python
date: 2018-01-06 10:10:48
tags: SHELL
categories: 脚本-测试-工具
keywords: SHELL
---
# wc 命令
wc命令 是为统计指定文件中的字节数,字数,行数,并将统计结果显示输出
-c 统计字节数。
-l 统计行数。
-m 统计字符数。这个标志不能与 -c 标志一起使用。
-w 统计字数。一个字被定义为由空白、跳格或换行字符分隔的字符串。
-L 打印最长行的长度。
-help 显示帮助信息
--version 显示版本信息

使用
```code
wc -l test.txt  统计一个文件有多少行
wc -w test.txt  统计字数
```
wc 经常和 cat ls命令一起使用
```code 
cat file | grep 'Error' wc -l  统计包含Error字符的 有多少行
ls -l | wc -l 统计这个目录下有多少文件
```


