---
title: JS-Promise对象
meta: JavaScript  
date: 2018-06-06 19:50:43
tags: JS
categories: JS
keywords: Promise
---

# Promise
>所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）

[阮一峰 ECMAScript6 入门](http://es6.ruanyifeng.com/#docs/promise)
  * 特点： 对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）

  * 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。


  * 缺点：首先，无法取消Promise，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。第三，当处于pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。
