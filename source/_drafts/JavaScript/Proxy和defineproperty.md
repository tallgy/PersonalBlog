---
title: Proxy和defineproperty
date: 2022-03-01 22:55:17
tags:
 - JavaScript
 - 随笔
categories:
 - JavaScript
 - 随笔
---



#  Proxy 和 defineproperty 的区别和使用

​		首先我们学习了 vue 的响应式之后就会知道，vue2 的响应式是根据 Object.defineproperty 进行的处理，通过编写里面的 getter 和 setter 方法。在getter 和 setter 方法里面进行了处理。getter 方法里面将对应的DOM加入了 dep数组，然后

* data数据通过使用 observer 转换成了 getter/setter 形式的来追踪变化。
* 当通过 watcher 读取到数据时，会触发到 getter 将其添加到 依赖中。
* 当数据发生变化时，会触发setter，然后想 dep 中的依赖（watcher） 发送通知。
* watcher 接收到通知之后，则会发送通知。然后便会触发视图的更新。



​		但是同时 Object.defineproperty 也有对应的缺点，比如这个是针对对象进行的监听，只能监听到对象的改变，而不能监听到数组的变化。

​		所以就有了 vue3 使用的 proxy，Proxy 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。proxy的特点是，可以深度监听，并且可以监听 新增和删除的属性。还可以监听数组的变化。



## Object.defineproperty

* 这个方法会直接 在一个对象上定义一个新属性，或者修改一个一个对象的现有属性，并返回此对象。

* ```
  const object1 = {};
  
  Object.defineProperty(object1, 'property1', {
    value: 42,
    writable: false
  });
  
  object1.property1 = 77;
  // throws an error in strict mode
  
  console.log(object1.property1);
  // expected output: 42
  ```

* Object.defineProperty(obj, prop, descriptor)

* 参数

  * obj： 要定义的对象
  * prop： 要定义或修改的属性的名称或者 Symbol
  * descriptor： 要定义或修改的属性描述符。

* 返回值： 被传递给函数的对象。

> 备注：在ES6中，由于 Symbol类型的特殊性，用Symbol类型的值来做对象的key与常规的定义或修改不同，而Object.defineProperty 是定义key为Symbol的属性的方法之一。



简单示例：

```
const obj = {
  a: 1,
}

Object.defineProperty(obj, 'b', {
  value: 2,
})

Object.defineProperty(obj, 'a', {
  get() {
    return 1;
  }
})

注意点：
数据描述符和存取描述符不能混合使用
```

### 数据描述符 和 存储描述符 

| `configurable` | `enumerable` | `value` | `writable` | `get`  | `set`  |        |
| -------------- | ------------ | ------- | ---------- | ------ | ------ | ------ |
| 数据描述符     | 可以         | 可以    | 可以       | 可以   | 不可以 | 不可以 |
| 存取描述符     | 可以         | 可以    | 不可以     | 不可以 | 可以   | 可以   |

​		简单来说，就是数据描述符和存储描述符是不能混用的。混用会报错。

* 数据描述符：value， writable
* 存取描述符： get， set



​		那么我们可以知道，因为数据描述符和存取描述符不能同时进行创建和使用。我们可以查看Vue2的源码，Vue2里面是通过 defineReactive 进行创建的响应式，一个 defineReactive 代表的就是一个value，内部会维护一个 Dep，通过 Dep.depend() 和 Dep.notify 来进行依赖添加和 通知。

```
observe(方法) ---()--->> Observer(类) ---( walk )--->> defineReactive(方法) ---(line:178)--->> observe

就是这样的一个递归形式，从 observe 里面通过判断 不是对象 或者 属于VNode来结束递归
```



## Proxy























