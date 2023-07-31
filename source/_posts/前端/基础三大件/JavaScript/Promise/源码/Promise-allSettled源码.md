---
title: Promise.allSettled源码
date: 2023-07-31 16:12:19
tags:
 - JavaScript
 - Promise
categories:
 - JavaScript
 - Promise
---

# Promise.allSettled 源码

## 使用方式

和 all 方法差不多，但是区别在于返回情况
不会 reject ，resolve 的返回值是一个对象，存在 status 属性

## 源码实现

源码分析

**首先是调用所有的 Promise**

```JavaScript
const resArr = [];
for (let i=0; i<len; i++) {
  const curPromise = promiseList[i];
  curPromise.then(re => {
    // 将其加入list里面
    resArr[i] = {
      status: 'fulfilled',
      value: re
    }
  }, rj => {
    resArr[i] = {
      status: 'rejected',
      value: rj
    }
  })
}
```


**需要考虑 promiseList 参数里面是否有非Pomise**

这个我们一般只要考虑到是一个 thenable 对象，说明
这个就是一个具有 Promise 状态的

```JavaScript
const isPromise = (obj) => {
  return typeof obj === 'object' && typeof obj.then === 'function';
}
```

**需要何时进行 resolve**

因为 allSettled 方法只会 resolve 出去，所以我们只需要在所有的
promise 都已经返回了，那么就可以 resolve 了

```JavaScript
count++;
if (count === len) {
  resolve(resArr)
}
```

**整体源码**

```JavaScript

const allSettled = (promiseList) => {
  const listLen = promiseList.length;
  let count = 0;

  const isPromise = (obj) => {
    return typeof obj === 'object' && typeof obj.then === 'function';
  }

  return new Promise((resolve, reject) => {
    const resArr = [];

    const addPromiseRes = (index, value, isResolve = true) => {
      count++;
      resArr[index] = {
        status: isResolve ? 'fulfilled' : 'rejected',
        value,
      }
      if (count === listLen) {
        resolve(resArr);
      }
    }
  

    for (let i=0; i<listLen; i++) {
      const curPromise = promiseList[i];
      if (isPromise(curPromise)) {
        curPromise.then(re => {
          addPromiseRes(i, re)
        }, rj => {
          addPromiseRes(i, rj, false)
        })
      } else {
        addPromiseRes(i, curPromise)
      }
    }
  })
}

```