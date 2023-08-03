---
title: 'call,apply,bind实现'
date: 2021-10-18 14:04:16
tags:
 - JavaScript
 - call
 - apply
 - bind
 - 源码
categories:
 - [JavaScript, Global_Objects]
 - [JavaScript, Global_Objects]
 - [JavaScript, Global_Objects]
---



#  call,apply,bind源码的实现

```

相同点：都是替换 this 的指向
区别：
  call：第一个参数为this指向，后续的参数当作本身方法的传参
  apply：第一个参数为this指向，第二参数为一个数组，数组内部为方法本身的传参
  bind：整体同 call 方法一样，但是首先，bind不会马上执行方法，其次，bind方法返回的方法也可以携带参数继续向原方法添加。

案例：

const a = (a, b) => {
    console.log(a, b);
}

// call
a.call({}, 1, 2);

// apply
a.apply({}, [1, 2]);

// bind
const b = a.bind({}, 1, 2);
b();

// OR

const b = a.bind({}, 1);
b();
b(2);

```

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

更新的写法
注意：不能使用箭头函数，因为箭头函数的this指向不会被改变。
```
const call = function(ctx, ...args) {
    const fn = Symbol('fn');

    ctx[fn] = this;
    const res = ctx[fn](...args);

    return res;
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

更新的写法
```
const apply = function(ctx, args) {
    const fn = Symbol('fn');

    ctx[fn] = this;
    const res = ctx[fn](...args);

    return res;
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

更新的写法
```JavaScript
const bind = function(ctx, ...args) {
    const fn = Symbol('fn');
    ctx[fn] = this;

    return (...newArgs) => {
        const res = ctx[fn](...args, ...newArgs);
        return res;
    }
}
```


```
fn.bind(db, 'cc', 1)();
```

