---
permalink: sidebar-tall-toc-test-post
title: Hexo 安装部署记录
date: 2018-01-03 18:46:14
tags: Hexo 
category: 脚本-测试-工具
keywords: 搭建博客,Hexo
meta: hexo 博客搭建
url: http://dreamqq.cn/:year/:month/:day/sidebar-tall-toc-test-post
author: 扆佳梁
---

# Hexo
Hexo 很早就听接触过,中间由于时间安排,以及自己太懒,就没有继续深入了解.
平时都使用印象笔记记录,随着工作时间越长,发现印象笔记用久了,整理的知识点都太碎片化了,也许是私有的缘故,就什么都往里塞,对知识点都没有系统化的整理.尤其是印象笔记搜索功能支持的不是很好.

还有一点搭建一个自己的博客也是在面试过程中看上去体面,我个人不是一个很擅长表达,也会描述自己做的事情好像有多难,自己有多厉害.也许你在项目中遇到很多意想不到的坑,搞得你头昏脑涨的,但是这程序你做过了,懂了,回头看,也就那样.所以,我也不清楚,这有多难?所以搞了一个博客让面试官可以多了解一点自己.

总之:
1.整理一些系统化的自己研究的成果分享上来.2.让自己在面试过程中比较顺利

## 安装 hexo 

``` bash
$ npm install -g hexo-cli 
```

### 初始化工作目录 这里三个命令需要连续执行 

``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

### 安装插件

``` bash
npm install hexo-deployer-git --save 发布git
npm install hexo-generator-feed --save
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
npm install hexo-generator-searchdb --save 支持本地搜索
```
搜索对于我来说是一个很重要的功能 [配置本地搜索](http://theme-next.iissnan.com/third-party-services.html#local-search)

## 写文章

#### 创建分类 标签 关于我
``` code
hexo new page "tags"
hexo new page "categories"
hexo new page "about"
```
上面命令会在source下创建文件的tags,categories,about目录,里面各有index.md
修改tags和categories 下的index.md 添加:`type: "categories"` 就会自动获取所有标签,分类
```
  title: All categories
  date: 2014-12-22 12:39:04
  type: "categories"
```
可以使用 http://ip/about/ 直接访问
### 写普通文章
``` code
hexo new "文章标题"
```
区别上面的创建 new page 
### 私密文章
``` code
$ hexo new draft "new draft"
```
会在source/_drafts目录下生成new draft.md文件,这个文件不会被显示在页面,链接也访问不到.

如果你希望强行预览草稿，更改配置文件：`render_drafts: true`

或者，如下方式启动server：
```code
$ hexo server --drafts
```


## 删除文章
直接删除即可，清除该文章生成的分类和标签请使用`hexo clear`命令重置，然后重新生成静态文件即可。


## 域名绑定
1.购买域名
2.ping yourname.github.io 获取地址
3.在万网配置解析域名 -> ip
4.在blog/source/ 目录创建新文件 CNAME 内容是`你的域名`
5.`hexo d -g` 重新生成 发布







