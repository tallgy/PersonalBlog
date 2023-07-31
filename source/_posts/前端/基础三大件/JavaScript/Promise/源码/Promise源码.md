---
title: Promise源码
date: 2023-07-31 17:21:25
tags:
 - JavaScript
 - Promise
categories:
 - JavaScript
 - Promise
---

# Promise 源码

Promise A+ 规范
```
https://www.jianshu.com/p/e0f91e03d6c1
```

## version 1

解决问题：
记录了三个状态 pending、fulfilled、rejected
保持了三个状态不会出现逆转情况
添加了 then 方法调用

问题：
并没有解决异步的问题
因为 then 方法会在链式调用上直接进行调用，此时还需要解决等待的问题

```JavaScript

class Promise1 {
  // 记录三个状态
  pendding = 'pending'
  fulfilled = 'fulfilled'
  rejected = 'rejected'
  // 默认状态
  status = this.pendding;
  // resolve reject 的值
  resValue = undefined;

  constructor(callback) {
    this.resolve = this.resolve.bind(this);
    this.reject = this.reject.bind(this);
    if (typeof callback !== 'function') {
      this.resolve(callback);
    } else {
      callback(this.resolve, this.reject)
    }
    return this;
  }

  resolve(value) {
    // 保持唯一性
    if (this.status !== this.pendding) {
      return;
    }
    this.resValue = value;
    this.status = this.fulfilled;
  }

  reject(value) {
    if (this.status !== this.pendding) {
      return;
    }
    this.resValue = value;
    this.status = this.rejected;
  }

  then(resolveCallback, rejectCallback) {
    if (this.status === this.pendding) {
      return;
    }

    if (this.status === this.fulfilled) {
      resolveCallback && resolveCallback(this.resValue)
    }
    if (this.status === this.rejectCallback) {
      rejectCallback && rejectCallback(this.resValue)
    }
  }
}

```


## 异步问题的解决办法

如果再调用 then 方法的时候，如果此时 的状态还是 pending
那么就会用一个栈去进行存储。在 resolve 的时候去调用栈。
用一个栈来将 then 方法去存储下来
然后在转变的时候通过 resolve 方法去调用 then 方法。






















