---
title: fetch发送2次请求的原因
date: 2021-11-06 13:20:43
tags:
 - 随笔
categories:
 - 随笔
---



#  fetch发送2次请求的原因

fetch发送post请求的时候，总是发送2次，第一次状态码是204，第二次才成功？

原因很简单，因为你用fetch的post请求的时候，导致fetch 第一次发送了一个Options请求，询问服务器是否支持修改的请求头，如果服务器支持，则在第二次中发送真正的请求。



## 先说产生的前提条件

* 请求不同源
* 不属于简单请求



## 请求不同源

同源的概念：

​		如果两个 URL 的 [protocol](https://developer.mozilla.org/zh-CN/docs/Glossary/Protocol)、[port (en-US)](https://developer.mozilla.org/en-US/docs/Glossary/Port) (如果有指定的话)和 [host](https://developer.mozilla.org/zh-CN/docs/Glossary/Host) 都相同的话，则这两个 URL 是*同源*。这个方案也被称为“协议/主机/端口元组”，或者直接是 “元组”。

所以简单来说，因为浏览器的安全策略，所以对于不同源的请求是不会成功的。



## 不属于简单请求

简单请求：

* 请求方法是以下三个方法：
  * HEAD, GET, POST
* HTTP的头信息不超出以下几种字段
  * Accept， Accept-Language， Content-Language， Last-Event-ID， Content-Type(只限于三个值`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`)

不满足以上两个条件就属于非简单请求



## 对于非简单请求，需要先进行预检请求(preflight)

​		浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的`XMLHttpRequest`请求，否则就报错。

​		204的状态表示了处理了请求，没有返回实体内容。

