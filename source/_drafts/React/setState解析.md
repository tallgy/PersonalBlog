---
title: setState解析
date: 2023-05-25 17:37:50
tags:
categories:
---


## 是同步还是异步

* setState并不是单纯异步或者是同步的，它的表现会因为调用场景的不同而不同
  * 在react钩子函数中也就是我们说的生命周期函数里是setState异步的，还有合成事件也就是我们定义的函数，setState也是异步的。
  * 而在setTimeout、setInterval等函数中，包括在DOM原生事件中，setState都是同步的。

原因

```
    导致这种差异的原因，是因为react的事务机制和更新机制的工作方式决定的。事务机制，在源码中这个变量为 isBatchingUpdates ，在执行react钩子函数和合成事件之前，这个变量都会被react修改成true,当这个变量为true时，setState就不会生效，当钩子函数或者合成事件执行完毕之后，这个变量会被设置为false,此时setState才会生效，isBatchingUpdates就好像一把锁，在isBatchingUpdates的约束下setState只能是异步的。但是当遇到setTimeout时，事情就会有点不同，isBatchingUpdates的约束对setTimeout内部的执行逻辑完全没有约束能力,这是因为setTimeout就是异步的，当异步函数开始执行的时候，同步任务早就结束了，isBatchingUpdates早就被设置为了false。批量更新，每来一个setState，就会把他塞进一个队列里面，最后再合并相同任务，最后只针对需要更新的state进行操作

```
