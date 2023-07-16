---
title: async和await的基本使用
date: 2021-11-07 10:34:28
tags:
 - JavaScript
 - Promise
categories:
 - JavaScript
 - Promise
---



#  async和await

​		简单来说就像是将一个异步转化为一个同步的操作。
  同时，async 和 await 更像是一个 基于 Promise 的 Generater 的语法糖。


## 简单的使用方式

```
async function name() {
	await xxx;
	
	return 3;
}
```

​		首先，await需要在 async之内，否则会报错，其次await等待的是一个promise对象，否则就没有什么效果。

**上面的函数的执行：**

* 先执行了async函数，创建了一个Promise，此时为pending阶段
* 然后进入函数内部，执行到了await部分。如果await后面接一个promise的行为，等待一个resolve的返回，async函数内部后面的一些方法不会执行，但是不会影响到外面的函数的继续 运行。
* 接收到了resolve，将其值返回。
* 然后继续运行，直到结束或return，有return x，则为resolve(x); 如果是返回一个promise方法，则获取其resolve或者reject进行执行。



​		先看看示例

```
async function test() {

  let a = await new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(1);
    }, 2000);
  })
  console.log(a);

  return 3;
}

test().then((res) => console.log(res));
console.log(111);
```

​		我们分析一下：

* 首先，可以知道a=1，test的resolve为3。
* 然后就是执行的顺序了，
* async函数执行，发现是一个promise的方法，直接先执行后续函数。然后再执行promise，这个是调度的机制
* async函数内部执行，虽然a后面是一个promise方法，但是使用了await，所以后面的会等待。
* 输出a，执行后面的，return 3
* return 3，就类似于了Promise.resolve(3)，所以test.then 可以调用。



## reject处理

​		我们同时也可以发现，如果在 async函数内部调用了reject，会直接抛出异常。

​		此时我们的处理方式，一般来说趋向于使用try，catch进行错误捕获，然后对错误进行处理。

对于错误的处理也有两种方式：

* ```
  async function test() {
  
    let a = await new Promise((resolve, reject) => {
      setTimeout(() => {
        reject(1);
      }, 100);
    }).catch(e => console.log('err'));
    console.log(a);
  
    return 123;
  }
  
  test()
  ```

* ```
    try {
      a = await new Promise((resolve, reject) => {
        setTimeout(() => {
          reject(1);
        }, 100);
      });
      console.log(a);
    } catch (e) {
      console.log(e + ' -- ');
      console.log('err');
    }
    
  输出：
  1 --
  err
  
  ```

注意点：

​	这里catch的e，是reject传递的值。



