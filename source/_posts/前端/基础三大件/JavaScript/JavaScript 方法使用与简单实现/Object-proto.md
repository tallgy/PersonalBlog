---
title: Object.__proto__
date: 2022-03-03 11:07:45
tags:
 - JavaScript
 - 随笔
categories:
 - JavaScript
 - 随笔
---



#  关于 属性原型的一个问题

​		先说问题和总结

```
const obj = {
  a: 1,
}
obj.__proto__.x = 1

// 可以这样直接访问
x.log
```

​		总结：

​		因为全局的上下文的 proto 也是 Object.prototype。所以我们修改了 obj.proto 实际上也修改了 Object.prototype。所以，不要随意修改高层的原型，容易出大问题啊。



​		首先我们可以知道，一个对象，存在一个 prototype 原型，然后在创建一个实例的时候，会将其创建为一个 proto 的内部属性进行赋值。

```
const obj = {
  a: 1,
}
obj.__proto__.x = 1
```

​		如果上面这样写，就是说，设置了 obj 的 proto 属性，然后就会在 Object.prototype 里面创建上这个属性，因为 proto 就是引用的 prototype，对于一个对象，他的引用就是 Object 的 prototype 了。

​		这个其实没啥，但是我们却可以通过一个例子发现一个问题。

```
const obj = {
  a: 1,
}
obj.__proto__.x = 1
console.log(x)
```

​		看看这里，我先创建一个对象 obj，然后对其原型创建一个x的属性，然后我们如果直接访问x，则会发现，还真的可以访问。。

​		并不会报错。这里的原因又是为什么呢？

* 简单来说就是，运行环境下，也是存在一个上下文的，存在上下文，那么便会有上层原型，JS里面所有的原型的最终的指向就是 Object.prototype 都会挨着挨着这样指上去。所以，一切的最终指向都是 顶层的Object的 prototype 。然后，就是运行环境的上下文问题了。

  * ```
    const obj = {
      a: 1,
    }
    obj.__proto__.x = 1
    console.log(x)
    function c() {}
    console.log(c.x)
    ```

* 运行环境的上下文，我们可以通过输出this，来知道上下文是什么，当然，输出一般都是一个空对象或者是window，但是这个并不是重点，重点是，为什么x，可以输出，因为他仅仅只是放在了obj的proto里面

* 我们可以知道，JS 的原型最终都是指向的 Object.prototype 所以，obj.proto 其实也是指向的 这个，因为他这个是顶层的一个 new Object，然后我们修改了 proto 就是修改了 prototype，所以 JS 环境下的 prototype 就被修改了，然后我们访问 x，然后，在本地上下文不存在，于是就会去原型进行查找，this.proto 就是 prototype，所以这样的指向最后就是 prototype了。

* ```
  console.log(x, this.__proto__, Object.prototype)
  this.__proto__ === Object.prototype ： true
  ```



