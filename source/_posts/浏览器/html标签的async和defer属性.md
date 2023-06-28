---
title: html标签的async和defer属性
date: 2023-6-27 16:11:12
tags:
 - HTML
 - 随笔
categories:
 - HTML
 - 随笔
---


  首先 HTML 的解析流程是从上往下的。
  同时在遇到 script 标签的时候，会产生阻塞，因为 script 可以对 dom 进行操作，所以需要先将 script 里面的代码进行执行，避免后续重新构建 DOM 结构。
  当然 script 标签 也会让 css 标签被阻塞，原因同上一样。。


所以在以前的时候，script 标签往往都写在最后面，这样会最小化的影响 DOM 的生成，但是本质来说，放在最后面，存在 script 标签，其实还是会让 DOM 的生成结束被影响。并没有在本质上提升了速度。

所以就使用到了 async 和 defer 这两个的属性就是，让浏览器知道，这个 script 标签里面的内容不会影响到 DOM 的生成，可以被执行。（为什么会这样说，因为使用了这两个属性之后，js的执行就不是绝对的位置了，所以需要保证 js 不会影响 DOM 的生成。


async 表示，尽快执行。  （https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/script#async）
defer 表示，下载完之后，在浏览器DOM解析完成之后，在出发 DOMContentLoaded 事件之前被执行。 （https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/script#defer）



![Alt text](./html%E6%A0%87%E7%AD%BE%E7%9A%84async%E5%92%8Cdefer%E5%B1%9E%E6%80%A7/image.png)


![Alt text](./html%E6%A0%87%E7%AD%BE%E7%9A%84async%E5%92%8Cdefer%E5%B1%9E%E6%80%A7/image1.png)

https://blog.csdn.net/weixin_42655717/article/details/120329272