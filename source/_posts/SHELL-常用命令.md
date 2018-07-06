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
# du 命令
du命令也是查看使用空间的，但是与df命令不同的是Linux du命令是对文件和目录磁盘使用的空间的查看，还是和df命令有一些区别的。
[参考](http://man.linuxde.net/du)
```code
-a或-all 显示目录中个别文件的大小。
-b或-bytes 显示目录或文件大小时，以byte为单位。
-c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
-k或--kilobytes 以KB(1024bytes)为单位输出。
-m或--megabytes 以MB为单位输出。
-s或--summarize 仅显示总计，只列出最后加总的值。
-h或--human-readable 以K，M，G为单位，提高信息的可读性。
-x或--one-file-xystem 以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
-L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小。
-S或--separate-dirs 显示个别目录的大小时，并不含其子目录的大小。
-X<文件>或--exclude-from=<文件> 在<文件>指定目录或文件。
--exclude=<目录或文件> 略过指定的目录或文件。
-D或--dereference-args 显示指定符号链接的源文件大小。
-H或--si 与-h参数相同，但是K，M，G是以1000为换算单位。
-l或--count-links 重复计算硬件链接的文件。
```
实例：
* du -kh Demo/  查看Demo目录下所有文件的大小，以kb为单位， 并且显示k 提高可读性
```code
8.0K	Demo/algorithm-monitor/.git/logs/refs/heads
24K	Demo/algorithm-monitor/.git/logs/refs
32K	Demo/algorithm-monitor/.git/logs
52K	Demo/algorithm-monitor/.git/hooks
8.0K	Demo/algorithm-monitor/.git/info
176K	Demo/algorithm-monitor/.git
4.0K	Demo/algorithm-monitor/.idea/inspectionProfiles
36K	Demo/algorithm-monitor/.idea
44K	Demo/algorithm-monitor/abcft_algorithm_monitor
272K	Demo/algorithm-monitor
24K	Demo/abcft_algorithm_monitor-0.1.1.dist-info
316K	Demo/

```

* du -sh
```code
yijialiang@yijialiang:~$ du -skh Demo/
316K	Demo/
```
# ln cp 命令
ln 源文件路径 链接目标路径

```code
ln test test2 表示为test 创建一个test2的硬连接
```
硬连接和拷贝的区别就是test test2 在内存中指向同一块空间，修改一个内容，另一个也会改变。

```code
ln -s test test2 表示为test 创建一个test2的软连接
```
1.test test2 在内存中指向同一块空间，修改一个内容，另一个也会改变。
2.当test 源文件被删除以后，test2也就失效了，硬连接则不会。


# Shell中判断语句if中-z至-d的意思
[shell if 语句](https://www.cnblogs.com/coffy/p/5748292.html)
```
[-z string] “string”的长度为零则为真
```
