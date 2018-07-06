---
title: git 使用记录
meta: ios,python
date: 2018-04-13 19:57:48
tags: git命令
categories: 脚本-测试-工具
keywords: git rebase amend
---

最近新换了工作，还在适应中，主要是代码习惯问题。
基本命令就不多说了。
#### git rebase
推荐几个参考的链接：

关于rebase 和 merage 的区别可以重点看下面第一个链接的这篇文章
总结下就是：rebase 代替合并
```code
$ git rebase branch-B
```
表示  将 分支B合并到当前分支
使用rebase 他们希望项目拥有一个单一的历史发展轨迹。比如一条直线。在历史纪录上**没有迹象表明在某些时间它被分成过多个分支。**

[这个图文并茂 rebase](https://www.git-tower.com/learn/git/ebook/cn/command-line/advanced-topics/rebase)

[rebase的中文文档](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E5%86%99%E5%8E%86%E5%8F%B2)

[Git提交历史的修改删除合并](https://juejin.im/post/5a30c1786fb9a045211eb218)

[压缩commit 提交](https://blog.csdn.net/itfootball/article/details/44154121)

>总结下使用心得：

目标是：最后合并的时候保持当前分支，只有一个commit。
(一般来说一个commit 一个更新内容。可以有多个commit)

##### 第一种情况 假如修改了当前的文件，还没有commit，
可以使用
```code
git commit --amend
```
表示提交文件，并修改最近一次的更新内容。并不会新增commit。

##### 第二种情况，假如我们已经提交了多个commit，想将多个commit合并成一个commit。

重点看下这篇文章

[Git提交历史的修改删除合并](https://juejin.im/post/5a30c1786fb9a045211eb218)

可以使用如下命令：
```code
git rebase -i head～2
```
表示准备将近2个commit合并
修改pick 为 squash
前一个就是上一个的意思。所以修改pick从下到上修改。

```
pick：简写p，启用该commit；
reword：简写r，使用该commit，但是修改提交信息，修改后可以继续编辑后面的提交信息；
edit：简写e，使用commit，停止合并该commit；
squash：简写s，使用该commit，并将该commit并入前一commit；
drop：简写d，移除该commit；
```
注意这里可能并不是 进入vi编辑器模式。而是进入nano编辑器模式。

[nano编辑器使用教程](https://www.vpser.net/manage/nano.html)

用的比较多的大概顺序就是：
```code
Ctrl+ O 保存
Enter 回车 确认
Ctrl + X 退出
Y 确认 N 不保存
```

#### Git push remote rejected {change ### closed}

经常--amend 修改commit 有可能报这个错误。

原因：
>I got the same message. And it was because I have managed to get the same Change-Id for two commits. Maybe due to some cherry-picking or similar between my local branches. Solved by removing the Change-Id from the commit message, a new Id then was added by the commit hook.

方法：
```code
git commit --amend
对 commit 内容 delete change id
save and quit
new change id will be added to the commit. it can be verified by git log.
push again
```

#### 我们使用的是Gerrit 管理项目
所以 不能直接push 需要先进入code review
提交到refs/for/{xx}分支，然后通过gerrit进行review之后merge, {xx}为对应的分支名称，例如：
```code
git push origin HEAD:refs/for/master
```
#### 版本回退
当本地已经提交了commit，想要撤销这个commit.
```
git reset --hard HEAD^
```

#### 多人开发提交之前
```
git pull --rebase
防止 别人提交了，冲突
```
