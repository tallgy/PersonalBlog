---
title: js沙箱
date: 2023-08-29 00:54:49
tags:
 - sandbox
categories:
 - sandbox
---

# JavaScript 沙箱 sandbox

## LegacySandbox proxy 


就是使用 proxy 然后创建一个代理
在set的时候会记录 快照 和 在 回滚的时候需要记录的值
proxy 需要是一个空对象，不能直接用 context 进行代理
因为这样会在 回滚和恢复的时候也触发代理，会有问题。

无法考虑到对象的修改

多次创建的同一个 context 的沙箱，修改会全部都影响。
所以在qiankun里面 只适合一次只启动一个应用。

```JavaScript
const createFakeWindow = (context = window) => {
  const fakeWindow = {};
  /** 新增的key */
  const addValue = new Set();
  /** 修改后记录的原始数据 */
  const modifyValue = {};
  /** 快照，用于回滚 */
  const snapValue = {};

  // 发现这个需要使用一个 fakeWindow 的空对象，
  // 否则在回滚的时候也会触发 set，所以为了避免触发
  // 需要使用这个去中间代理
  const proxy = new Proxy(fakeWindow, {
    set: function (_, p, newValue) {
      // 表示是存在的
      if (p in context) {
        if (!modifyValue[p]) {
          modifyValue[p] = context[p];
        }
      } else {
        // 表示没有存在的
        addValue.add(p);
      }
      snapValue[p] = newValue;
      context[p] = newValue;
    },

    get: function(_, p) {
      const value = context[p];
      if (typeof value === 'function') {
        // 解决方法的this指向问题
        // 需要注意，此时返回的方法已经不是原来的方法了。
        return Function.prototype.bind.call(value, context);
      }

      return value;
    }
  })

  // 回滚操作
  const inActive = () => {
    for (const value of addValue) {
      delete context[value];
    }
    for (const key in modifyValue) {
      context[key] = modifyValue[key];
    }
  }

  const active = () => {
    for (const key in snapValue) {
      context[key] = snapValue[key];
    }
  }

  return {
    proxy,
    inActive,
    active
  };
}

```


简单案例使用
```JavaScript


const obj = {
  a: 1,
  b: 3
}
const {proxy, inActive, active} = createFakeWindow(obj);

proxy.fn2 = function() {
  console.log('fn2 ', this);
}

proxy.c = 3;

console.log(obj);
inActive();
console.log('inactive ', obj);
active();
console.log('active ', obj);


proxy.fn2();


/*

{ a: 1, b: 3, fn2: [Function (anonymous)], c: 3 }
inactive  { a: 1, b: 3 }
active  { a: 1, b: 3, fn2: [Function (anonymous)], c: 3 }
fn2  { a: 1, b: 3, fn2: [Function (anonymous)], c: 3 }

*/

```


无法实现多个实例同时处理案例
多个实例会同时影响全局对象
```JavaScript

const obj = {
  a: 1,
  b: 3
}
// 注意这里变成了数组，并不是失误，而是因为笔主比较懒，所以把 createFakeWindow 返回对象
// 变成了返回数组，这样重命名比较方便。
const [proxy, inActive, active] = createFakeWindow(obj);
const [p, i, a] = createFakeWindow(obj);

proxy.fn2 = function() {
  console.log('fn2 ', this);
}

proxy.c = 1;
p.d = 3;
console.log(obj);

inActive();
console.log(obj);

/*

{ a: 1, b: 3, fn2: [Function (anonymous)], c: 1, d: 3 }
{ a: 1, b: 3, d: 3 }

*/

```


## ProxySandbox proxy

上面那个的最大问题是，我们全局同时只能有一个全局代理，多余的不会有效果
所以，我们如果有一个在单页面多微应用的情况，就需要重新思考了


