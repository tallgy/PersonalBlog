---
title: single-spa-leaked-globals
date: 2023-08-19 23:43:43
tags:
 - single-spa-leaked-globals
categories:
 - single-spa-leaked-globals
---

# single-spa-leaked-globals

实现的是一个页面只有一个子应用的 js 隔离

实现方式是 创建 快照模式。

globalVariableNames： 数组，里面是需要存储的属性


* bootstrap 的时候 将 window 里面的 globalVariableNames 属性放在 opts 的 capturedGlobals
* mount 的时候，直接将 window 的 capturedGlobals 里面的 globalVariableNames 属性，赋值入 window 。
* unmount 的时候，再删除 window 里面的 globalVariableNames 属性。


js 隔离首先需要自己确定被隔离的属性。如果没有将需要被隔离的属性放在数组里面，那么这个属性是不会被隔离的
其次，这个只是一个浅拷贝的隔离，就是说对于对象的修改无法解决
这个思路其实就是，会将 数组 里面的属性存下来，然后在移除的时候再删除掉
所以在 app1 上的属性， app2 就访问不上了。

其实是让数组里面的值，在每次回来的时候变成初始状态
这里对于最开始就存在于window里面的两个 app 的相同属性可以进行保存
但是对于一个是初始化的，一个是后面添加的相同属性会有问题

问题：
其实有点没有理解，从代码上来说，看起来像是 只是将 window 的部分属性放进了 opts 里，
然后再 mount、unmount 的时候，再进行更新赋值。
没有感觉到有什么作用，因为首先，赋值只是浅拷贝，
其次window还是可以被修改。

看来别人说，应该是在子应用添加的属性都会被记录下来，所以做到了 隔离

这里还是存在问题。

https://single-spa.js.org/docs/ecosystem-leaked-globals/