---
title: JavaScript定时器
date: 2022-01-22 19:28:13
tags:
 - JavaScript
 - 随笔
categories:
 - JavaScript
 - 随笔
---



#  JavaScript 定时器

这里就简单的提几句。常用的两个

```
setTimeout
clearTimeout

setInterval
clearInterval
```



## settimeout在执行之后，timer也不会清空的

都会执行，代表了，执行之后，timer不清空

```
let timer = settimeout(() => {}, 1)

然后再延时执行下面方法
log.timer
if (timer) log.1
timer && log.2
```



## cleartimeout清除之后 timer依然判断为true

```
cleartimeout(timer)

然后再延时执行下面方法, 同上
log.timer
if (timer) log.1
timer && log.2
```

