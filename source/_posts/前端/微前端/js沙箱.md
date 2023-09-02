---
title: js沙箱
date: 2023-08-29 00:54:49
tags:
 - sandbox
categories:
 - sandbox
---

# JavaScript 沙箱 sandbox

## LegacySandbox proxy 单实例沙箱


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


## ProxySandbox proxy 多实例沙箱

上面那个的最大问题是，我们全局同时只能有一个全局代理，多余的不会有效果
所以，我们如果有一个在单页面多微应用的情况，就需要重新思考了


通过 crate 创建，会放在原型，
好处：在方法调用的时候可以得到 context 的数据
缺点：还是可以直接修改到 window 里面的数据： 
  fakeWindow.a.c = 3; 如果 a 在 fakeWindow 中不存在，就会直接返回 window的数据
```JavaScript

const createFakeWindow = (context = window) => {
  // 通过这个，可以让方法在调用的时候，解决同时使用了 context 和 fakeWindow 中的属性
  const fakeWindow = Object.create(context);

  // 发现这个需要使用一个 fakeWindow 的空对象，
  // 否则在回滚的时候也会触发 set，所以为了避免触发
  // 需要使用这个去中间代理
  const proxy = new Proxy(fakeWindow, {
    set: function (target, p, newValue) {
      target[p] = newValue;
      return true;
    },

    get: function(target, p) {
      if (p === '__proto__') {
        // 这个属性返回的是 context
        // 如果直接返回就会出现可以直接通过内部修改外部的数据
        // 也可以不做考虑 ?
        return {};
      }

      return target[p];
    }
  })

  // 不需要对数据做任何处理
  const inActive = () => {
  }

  const active = () => {
  }

  return [
    proxy,
    inActive,
    active
  ];
}

```

使用案例
```JavaScript

const obj = {
  a: {
    c: 1,
  },
  b: 3
}
obj.fn2 = function() {
  console.log('fn2 ', this.b, this.c);
}

const [proxy, inActive, active] = createFakeWindow(obj);

proxy.c = 123;
proxy.fn2();

/*
可以同时获取到 proxy 的 和 obj 的数据
fn2  3 123
*/
```

当然，我们也可以创建一个 null 的对象，然后对于方法和属性的获取多层考虑
缺点在于 fakeWindow 和 window 是一个完全隔离的两个数据，所以方法的绑定this
无法解决同时在两边同时引用的问题。


## SnapshotSandbox 快照沙箱

如果没有 proxy 方法。那么我们可以使用快照沙箱的模式。

原理也比较简单，就是在 active 将所有的值保存下来，同时恢复数据
然后在 inactive 的时候，判断存在的 key 是否有变化，同时判断是否有新增的，然后再将所有 原来的值赋值过来
```JavaScript

const createFakeWindow = (context = window) => {
  const original = {};
  const modify = {};
  const proxy = context;

  // 不需要对数据做任何处理
  const inActive = () => {
    const originlalKeys = Object.getOwnPropertyNames(original);
    Object.getOwnPropertyNames(context).forEach(key => {
      // 记录变化，如果值不等，那么记录为变化
      if (original[key] !== context[key]) {
        modify[key] = context[key];
      }
      // 如果原来没有的，直接删除
      if (!originlalKeys.includes(key)) {
        modify[key] = context[key];
        delete context[key];
      }
    });

    // 同时将 original 的值都赋值给 context
    Object.getOwnPropertyNames(original).forEach(key => {
      context[key] = original[key];
    })
  }

  const active = () => {
    Object.getOwnPropertyNames(context).forEach(key => {
      original[key] = context[key];
    })

    Object.getOwnPropertyNames(modify).forEach(key => {
      context[key] = modify[key];
    })
  }

  return [
    proxy,
    inActive,
    active
  ];
}

```


```JavaScript

const obj = {
  a: {
    c: 1,
  },
  b: 3
}
obj.fn2 = function() {
  console.log('fn2 ', this.b, this.c);
}

const [proxy, inActive, active] = createFakeWindow(obj);
console.log('begin ', obj);
active();
proxy.c = 123;
console.log('active ', obj);
inActive();
console.log('inactive ', obj);

/*

begin  { a: { c: 1 }, b: 3, fn2: [Function (anonymous)] }
active  { a: { c: 1 }, b: 3, fn2: [Function (anonymous)], c: 123 }
inactive  { a: { c: 1 }, b: 3, fn2: [Function (anonymous)] }

*/
```
