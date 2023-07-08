---
title: WebWorker
date: 2021-11-06 14:49:25
tags:
 - JavaScript
 - WebWorker
 - 随笔
categories:
 - 随笔
---



#  WebWorker

参考文章

```
http://www.ruanyifeng.com/blog/2018/07/web-worker.html
https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API
```



## 是什么

​		简单来说就是为JavaScript提供了多线程。一些计算密集型或高延迟的任务，被 Worker 线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。



**注意点**：

* 同源的限制
* 无法使用DOM对象，document，window，parent对象，可以使用navigator 和 location 对象。
* 和主线程通过消息进行通信
* 不能使用 alert 和 confirm 方法，但是可以使用 XMLHttpRequest 对象发出 AJAX 请求。
* 无法读取本地文件。



**用法**：

使用 new 调用 worker 函数，创建一个线程

```
const worker = new Worker('xx.js');
```

​		参数是一个脚本文件，这个文件必须来自网络



### 主线程：

主线程通过 worker.postMessage() 向Worker发送消息

```
worker.postMessage('Hello World');
worker.postMessage({method: 'echo', args: ['Work']});
```

​		参数就是传给worker的数据



主线程通过 worker.onmessage 监听函数，接收子线程发回的消息

```
worker.onmessage = function (event) {
	event.data.log;
}
```

​		事件对象的data属性可以获取 worker发来的数据



主线程关闭worker线程

```
worker.terminate();
```



### Worker线程

worker线程需要一个监听函数，监听 message 事件

```
self.addEventListener('message', function(e) {
	self.postMessage('said: ', e.data);
})
```

​		`self`代表子线程自身，即子线程的全局对象

所以等同于 this. 和 直接创建

```
this.addEventListener();
addEventListener();
```

​		也可以使用 self.onmessage 指定

```
self.onmessage = function (e) {
	self.postMessage('said: ', e.data);
}
```

​		监听函数的参数是一个事件对象，data属性是主线程发来的数据



self.postMessage() 方法用来向主线程发送消息



self.close() 用于在Worker内部关闭自身。



# API

## 创建API

```
const worker = new Worker(jsUrl, options);

var myWorker = new Worker('worker.js', { name : 'myWorker' });
```

​		jsUrl,脚本网址,遵守同源策略，必须且是js脚本

​		options是一个配置对象，其中一个作用就是指定worker名称用来区分线程。



## 主线程使用API

```
Worker.onerror
	指定 error 事件的监听函数。
Worker.onmessage
	指定 message 事件的监听函数，发送过来的数据在Event.data属性中。
Worker.onmessageerror
	指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
Worker.postMessage()
	向 Worker 线程发送消息。
Worker.terminate()
	立即终止 Worker 线程。
```



## Worker线程使用API

```
self.name
	Worker 的名字。该属性只读，由构造函数指定。
self.onmessage
	指定message事件的监听函数。
self.onmessageerror
	指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
self.close()
	关闭 Worker 线程。
self.postMessage()
	向产生这个 Worker 线程发送消息。
self.importScripts()
	加载 JS 脚本。
```

