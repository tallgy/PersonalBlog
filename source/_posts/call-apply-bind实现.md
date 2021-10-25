---
title: 'call,apply,bind实现'
date: 2021-10-18 14:04:16
tags:
 - JavaScript
 - call
 - apply
 - bind
 - 源码
---



#  call,apply,bind源码的实现



## call方法

使用，就是替换了this的指向，传参使用的基本传参。

```
这里将方法写在了Function函数的原型里面，这样因为通过原型链就可以找到这个方法了。
Function.prototype.myCall = function (ctx) {
  ctx = ctx || window;

  //这里这个this，指向的就是方法。这里使用Symbol，用于避免出现方法重复。
  const fn = Symbol('fn');
  ctx[fn] = this;

  // 获取参数
  const args = [...arguments].slice(1);
  // 使用ctx来进行调用。替换了this指向。
  const result = ctx[fn](...args);

  delete ctx[fn];
  return result;
}
```

```
fn.call(db, 'cc', 1);
```



## apply方法

同上，传参使用的是数组传参。

```
Function.prototype.myApply = function (ctx, args) {
  ctx = ctx || window;
  
  const fn = Symbol('fn');
  ctx[fn] = this;
  
  const result = ctx[fn](...args);
  
  delete ctx[fn];
  return result;
}
```

```
fn.apply(db, ['cc', 1]);
```



## bind方法

同call方法的参数，但是不是马上执行，需要自己执行一次

```
Function.prototype.myBind = function (ctx) {
  ctx = ctx || window;
  
  const fn = Symbol('fn');
  const args = [...arguments].slice(1);
  
  ctx[fn] = this;
  
  return () => {
    const result = ctx[fn](...args);
    delete ctx[fn];
    
    return result;
  }
}
```

```
fn.bind(db, 'cc', 1)();
```

