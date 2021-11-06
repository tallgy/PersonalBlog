---
title: Cookie、SessionStorage、LocalStorage的区别
date: 2021-11-06 13:35:43
tags:
 - 随笔
 - Cookie
 - LocalStorage
 - SessionStorage
categories:
 - 随笔
---



#  Cookie、SessionStorage、LocalStorage的区别

**共同点：**

* 保存于浏览器端
* 属于同源



## Cookie

* 数据始终在同源的HTTP请求中携带，即使不需要。
* 存在路径的概念，可以限制cookie只属于某个路径。
* 存储的大小只有4k左右。
* 一般由服务器生成，设置过期时间，如果不设置过期时间，则表示这个cookie生命周期为浏览器会话期间，只要关闭浏览器窗口，cookie就消失了。

使用：

```
JavaScript
	document.cookie = 'key=value';
	
HTTP响应头的 set-cookie
```



### 安全机制

* 响应头中setCookie设置HttpOnly 使JavaScript无法进行获取。
* 响应头设置 secure，告诉浏览器仅在HTTPS请求发送cookie
* 人为的设置时间，以及对key和value进行一些随机的生成。



## SessionStorage

* 存储时间：浏览器窗口关闭前有效，就是一个标签页。在标签页中，进行刷新也不会消失，但是关闭就会消失。
* 即使是同域名下的页面，只要不在同一浏览器窗口打开，那么他们的SessionStorage无法共享。
* 大小限制 5M
* 不和服务器进行通信，仅客户端使用

**使用**：

```
sessionStorage.setItem(key, value);
sessionStorage.getItem(key);
```



## LocalStorage

* 存储时间：永久有效。用作持久数据
* 同源的页面可以访问，不同于SessionStorage。
* 其他基本和SessionStorage相同。
* 基于上面的特点，LocalStorage可以作为浏览器的本地缓存方案，用来提升网页首屏渲染速度（根据第一请求返回时，将一些不变的信息直接存储在本地）

**使用**：

```
localStorage.setItem(key, value);
localStorage.getItem(key);
```

