---
title: JavaScript获取控制台的输入
date: 2022-03-13 20:51:51
tags:
 - JavaScript
 - 随笔
categories:
 - JavaScript
 - 随笔
---

#  JavaScript如何获取控制台的输入

​		首先我们知道 JavaScript 是一个用于浏览器的一个脚本语言，但是因为 Node.js 的开发，JavaScript 也组件用于服务器开发。所以便有了从控制台获取输入的一个方法。

​		当然第一次需要从控制台里面获取输入还是在笔试的时候发现的，大部分的笔试题是一个ACM的形式，而leetcode是一个核心代码的形式。

​		所以在笔试里面就需要获取输入了。



## readline 模块

​		简单来说就是，我们使用 readline模块来提供一个可读流。

​		一个简单的案例：

```
const rl = require('readline').createInterface({
  input: process.stdin,
  output: process.stdout
})

rl.on('line', (input) => {
  const index = input.lastIndexOf('/')

  console.log(input.slice(index+1))
  rl.close();
})
```

​		那么此时我们就可以获取到了控制台的输入了。

​		`readline.Interface` 类的实例是使用 `readline.createInterface()` 方法构造的。

​		这里 rl.on('line')，代表的是interface的line事件。line事件就是在 \n、\r时会触发的事件，通常是用户按回车或者返回时。

​		当然还有其他事件，我们这里就取常用的 line事件，同时在加上一个close方法。调用这个方法将会停止可读流

```
rl.close()，
```

​		对于其他的方式我们就查看文档即可。这里不做过多的解释。

```
http://nodejs.cn/api/readline.html
```

