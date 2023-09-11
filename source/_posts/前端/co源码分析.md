---
title: co源码分析
date: 2023-09-11 23:57:58
tags:
 - co
categories:
 - co
---

# co 源码分析


co模块简单使用
特点是，类似于一个 async await 的实现
```JavaScript
co(function *(){
  // resolve multiple promises in parallel
  var a = Promise.resolve(1);
  var b = new Promise((res) => {
    setTimeout(() => {
      res(2)
    }, 20)
  });
  var c = Promise.resolve(3);
  var res = yield [a, b, c];
  console.log('执行res', res);
  return res;
  // => [1, 2, 3]
}).then(res => {
  console.log('rrrr', res);
});
```

## 源码分析

其实也很简单就是
先用 生成器函数调用 next 获取 ret值
第一次调用 yield 然后取出值，将其作为 promise 调用，然后将最后的返回值 作为下次 next调用的参数
这里 next 参数，是会成为返回的值
比如：
const res = yield a        next(123)
那么这里的 res 就会变成 123

同时这里还有一个点就是
function() {
  yield 1;
} 
next(); 的第一次调用 虽然 yield 只有一个，但是 done还是false
需要两次调用 next

```JavaScript
function co(gen) {
  var args = slice.call(arguments, 1);

  return new Promise(function(resolve, reject) {
    gen = gen(args);

    onFulfilled();

    function onFulfilled(res) {
      var ret;
      ret = gen.next(res);
      next(ret);
      return null;
    }

    function next(ret) {
      if (ret.done) return resolve(ret.value);
      var value = toPromise(ret.value);
      if (value && isPromise(value)) return value.then(onFulfilled);
    }
  });
}

function toPromise(obj) {
  if (isPromise(obj)) return obj;
  if (Array.isArray(obj)) return Promise.all(obj);
  return obj;
}

function isPromise(obj) {
  return 'function' == typeof obj.then;
}
```
