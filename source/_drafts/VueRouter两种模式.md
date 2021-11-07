---
title: VueRouter两种模式
date: 2021-11-07 13:33:21
tags:
 - Vue
 - VueRouter
categories: 
 - Vue
 - VueRouter
---



#  VueRouter的两种模式

有history 和 hash两个模式





## history模式

在这里我先直接说一下history模式的问题，简单来说就是不能刷新页面，因为后端没有对其进行一个处理。

这里开始我想了很久都没有理解是什么意思，并且在本地访问也没有任何问题，后面才知道，这个是需要发布的时候，正式访问才会出现的问题。

首先，history模式在本地是没有问题的，只有在打包之后，运行那个dist文件夹服务才会发现

其次，问题的复现很简单，

* 先运行**打包**之后的项目，然后访问，注意，一定要打包之后的，如果直接运行server，你不会发现这个问题。
* 此时一般都是 `http://localhost:3000` 这种，访问之后，url一般都会根据前端的路由进行一次变化 `http://localhost:3000/bookkeeping` 我这里就变成这样了。
* 然后此时如果你刷新，请求的就是 `http://localhost:3000/bookkeeping` 而不是最开始那个，如果你没有对这个进行一个处理，那么就会404.

解决方法，简单来说就是对没有的域名都进行一个跳转为index.html页面的情况。



*  **方式1**：

  这个方式的前提条件就是 前端和后端的项目是合并的。并不是一个分离式的开发。此时的话，我们也可以知道，前端的页面请求是先通过了后端，然后后端进行配置，对于一些没有的路径进行一个处理即可

*  **方式2**：

  前后端进行了分离，这个时候就对前端运行就好了。下面这个是Vue的一个例子，我们在dist文件里新建一个js文件作为一个启动文件。然后在这里进行了一个文件的拦截，就像是一种虽然前后端分离，但是实则还是有一个后端。

  ```
  const http = require('http')
  const fs = require('fs')
  const httpPort = 80
  
  http.createServer((req, res) => {
    fs.readFile('index.html', 'utf-8', (err, content) => {
      if (err) {
        console.log('We cannot open "index.html" file.')
      }
  
      res.writeHead(200, {
        'Content-Type': 'text/html; charset=utf-8'
      })
  
      res.end(content)
    })
  }).listen(httpPort, () => {
    console.log('Server listening on: http://localhost:%s', httpPort)
  })
  ```

*  **方式3**：

  使用nginx进行代理。这里我没有成功，不知道是哪的问题，大致好像是使用 rewired进行重定向操作。具体以后再说。





## hash模式