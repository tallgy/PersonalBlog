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

```
// 注意: defineProperty 的get 方法是可以重复进行定义的。

const property = Object.getOwnPropertyDescriptor(obj, key);

const getter = property.get

Object.defineProperty(obj, key, {
	get() {
		const value = getter.call(obj)
		
		// 处理响应式
		...
		
		return value
	}
})
```

```
// 同时记住，不能这样写，会出现，无限递归的情况，vue2 的写法是，将getter方法取出来，直接运行getter方法。
Object.defineProperty(o, 'a', {
  get() {
    console.log('q')
    return o['a']
  }
})
```



## Proxy

​		**Proxy** 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。

语法

```
const p = new Proxy(target, handler)
```

### Proxy.revocable

​		与此对应的还有一个 Proxy.revocable 方法，它的使用和 new Proxy 基本一样，但是它的返回是一个包含了 代理对象本身和它的撤销方法的可撤销 Proxy 对象。

```
Proxy.revocable(target, handler)

返回一个对象，包含了 proxy(代理对象本身) 和 revoke(撤销方法)
```

​		注意：

​		一旦某个代理对象被撤销，它将变得几乎完全不可调用，在它身上执行任何的**可代理操作**都会抛出 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError) 异常（注意，可代理操作一共有 [`14 种`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy#methods_of_the_handler_object)，执行这 14 种操作以外的操作不会抛出异常）。一旦被撤销，这个代理对象便不可能被直接恢复到原来的状态，同时和它关联的**目标对象**以及**处理器对象**都有可能被垃圾回收掉。再次调用撤销方法 `revoke()` 则不会有任何效果，但也不会报错。

```
// 定义的 handler，这里定义了一个get方法，如果 prop 不存在于
const handler = {
    get: function(obj, prop) {
        return prop in obj ? obj[prop] : 37;
    }
};

const p = new Proxy({}, handler);
p.a = 1;
p.b = undefined;

console.log(p.a, p.b);      // 1, undefined
console.log('c' in p, p.c); // false, 37


使用 revocable 进行创建一个可撤销的Proxy代理。
const revocable = Proxy.revocable({}, handler)
const proxy = revocable.proxy
revocable.revoke();

console.log(proxy.foo); // 抛出 TypeError
proxy.foo = 1           // 还是 TypeError
delete proxy.foo;       // 又是 TypeError
typeof proxy            // "object"，因为 typeof 不属于可代理操作
```

​		和 Object.defineproperty 一样创建多个get方法的情况

```
const obj = {}

let p = new Proxy(obj, {
  get(target, prop) {
    console.log(target)
    console.log(prop)
    return target[prop]
  }
})

p = new Proxy(p, {
  get(target, prop) {
    console.log('target', target)
    console.log('prop', prop)
    return target[prop]
  }
})
```

​		通过 Proxy 和 Object.defineproperty 的使用，我们可以知道几个不同的地方

* Proxy的get方法带有参数，第一个是target，第二个prop。有的携带第三个参数newValue，defineproperty：不携带参数。
* Proxy的get方法里面，target[prop]，不会再次触发get方法。defineproprety里面如果再次使用 target[prop] 会多次触发get方法。



### Proxy 的方法

#### handler.apply() 

**`handler.apply()`** 方法用于拦截函数的调用。

```
var p = new Proxy(target, {
  apply: function(target, thisArg, argumentsList) {
  }
});
```

参数：

* target: 目标对象
* thisArg： 被调用时的上下文对象
* argumentsList： 被调用时的参数数组

返回值：

* 可以返回任何值

返回值就是显示的内容

拦截：

- `proxy(...args)`
- [`Function.prototype.apply()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 和 [`Function.prototype.call()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
- [`Reflect.apply()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/apply)

约束：

​		如果违反了以下约束，代理将抛出一个TypeError：

​		`target`必须是可被调用的。也就是说，它必须是一个函数对象。



#### handler.construct()

​		`**handler.construct()**` 方法用于拦截[`new`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new) 操作符. 为了使new操作符在生成的Proxy对象上生效，用于初始化代理的目标对象自身必须具有[[Construct]]内部方法（即 `new target` 必须是有效的）。



#### handler.defineProperty()

​		**`handler.defineProperty()`** 用于拦截对对象的 [`Object.defineProperty()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 操作。



#### handler.deleteProperty()

​		**`handler.deleteProperty()`** 方法用于拦截对对象属性的 [`delete`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/delete) 操作。



#### handler.get()

​		**`handler.get()`** 方法用于拦截对象的读取属性操作。

​		我们目前就先了解这个，因为这个就是vue3的架构里面使用的响应式的原理。

语法:

```
var p = new Proxy(target, {
  get: function(target, property, receiver) {
  }
});
```

参数:

* target：目标对象
* property：被获取的属性名
* receiver：Proxy或者继承Proxy的对象

返回值:

​		可以返回任何值。

拦截：

​		该方法会拦截目标对象的以下操作:

- 访问属性: `proxy[foo]和` `proxy.bar`

- 访问原型链上的属性: `Object.create(proxy)[foo]`

- [`Reflect.get()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/get)

  - `**Reflect**`**`.get()`**方法与从 对象 (`target[propertyKey]`) 中读取属性类似，但它是通过一个函数执行来操作的。

  - ```
    // Object
    var obj = { x: 1, y: 2 };
    Reflect.get(obj, "x"); // 1
    
    // Array
    Reflect.get(["zero", "one"], 1); // "one"
    
    // Proxy with a get handler
    var x = {p: 1};
    var obj = new Proxy(x, {
      get(t, k, r) { return k + "bar"; }
    });
    Reflect.get(obj, "foo"); // "foobar"
    ```

约束：

​		如果违背了以下的约束，proxy会抛出 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError):

- 如果要访问的目标属性是不可写以及不可配置的，则返回的值必须与该目标属性的值相同。

  - 意思就是说，对于 writeable 和 configurable 为 false，则返回值必须要相等，否则报错。

  - ```
    Object.defineProperty(obj, 'b', {
      value: 2,
      writable: false
    })
    
    let p = new Proxy(obj, {
      get(target, prop) {
        return 21;
      }
    })
    
    p.b 会报错， 因为返回值不相等，但是如果返回值改为相等就不会报错。
    ```

    

- 如果要访问的目标属性没有配置访问方法，即get方法是undefined的，则返回值必须为undefined。

  - 大概意思是不可访问的情况下，访问的get返回需要为 undefined。



#### handler.getOwnPropertyDescriptor()

​		**`handler.getOwnPropertyDescriptor()`** 方法是 [`Object.getOwnPropertyDescriptor()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor) 的钩子。

​		**`Object.getOwnPropertyDescriptor()`** 方法返回指定对象上一个自有属性对应的属性描述符。（自有属性指的是直接赋予该对象的属性，不需要从原型链上进行查找的属性）

语法：

```
var p = new Proxy(target, {
  getOwnPropertyDescriptor: function(target, prop) {
  }
});
```

拦截：

​		这个陷阱可以拦截这些操作：

- [`Object.getOwnPropertyDescriptor()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)
- [`Reflect.getOwnPropertyDescriptor()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/getOwnPropertyDescriptor)



#### handler.getPrototypeOf()

​		**`handler.getPrototypeOf()`** 是一个代理（Proxy）方法，当读取代理对象的原型时，该方法就会被调用。

​		Object.getPrototypeOf， 返回的是参数的原型。

```
const handler = {
  getPrototypeOf(target) {
    return monsterPrototype;
  }
};

简单来说，返回值就是原型。
```



#### handler.has()

​		 **`handler.has()`** 方法是针对 [`in`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/in) 操作符的代理方法。

​		如果指定的属性在指定的对象或其原型链中，则**`in` 运算符**返回`true`。



#### handler.isExtensible()

​		**handler.isExtensible()** 方法用于拦截对对象的Object.isExtensible()。

​		`**Object.isExtensible()**` 方法判断一个对象是否是可扩展的（是否可以在它上面添加新的属性）。



#### handler.ownKeys()

​		**`handler.ownKeys()`** 方法用于拦截 [`Reflect.ownKeys()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/ownKeys).

​		静态方法 `**Reflect**`**`.ownKeys()`** 返回一个由目标对象自身的属性键组成的数组。

描述：

​		Reflect.ownKeys` 方法返回一个由目标对象自身的属性键组成的数组。它的返回值等同于`Object.getOwnPropertyNames(target).concat(Object.getOwnPropertySymbols(target))。

​		getOwnPropertyNames：返回自身属性，包括不可枚举，但不包括 Symbol 属性

​		getOwnPropertySymbols：返回自身的 Symbol 属性



#### handler.preventExtensions()

​		**`handler.preventExtensions()`** 方法用于设置对[`Object.preventExtensions()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions)的拦截

​		`**Object.preventExtensions()**`方法让一个对象变的不可扩展，也就是永远不能再添加新的属性。



#### handler.set()

​		`**handler.set()**` 方法是设置属性值操作的捕获器。

语法：

```
const p = new Proxy(target, {
  set: function(target, property, value, receiver) {
  }
});
```

参数：

​		以下是传递给 `set()` 方法的参数。`this` 绑定在 handler 对象上。

* target： 目标对象

* property： 被设置的属性名或者 Symbol

* value： 新的属性值

* receiver： 最初被调用的对象。通常是 proxy 本身，但 handler 的 set 方法也有可能在原型链上，或以其他方式被间接地调用（因此不一定是 proxy 本身）。

* ```
  const monster1 = { eyeCount: 4 };
  
  const m = {
    a: 12,
  }
  const handler2 = {
    set(o, p, v, re) {
      // console.log(this)
      // console.log(re)
  
      return true
    },
    a: 2,
  }
  const pro2 = new Proxy(m, handler2)
  
  monster1.__proto__ = pro2
  
  // ok
  monster1.d = 123
  // not ok
  monster1.b = 321
  monster1.c = 321
  
  console.log(Object.getOwnPropertyNames(monster1), monster1.b)
  console.log(Object.getOwnPropertyNames(pro2), pro2.b)
  
  // 不理解为什么，不行
  // 目前来说，就是 children 的原型的指向是一个 proxy的实例，然后proxy的实例存在一个set方法。然后再 children设置属性的时候，出现了children不含这个属性，所以就会出现，调用proxy实例，方法。但是我返回值为true，但是还是没有赋值成功。但从案例来说，就是在原型修改之前和之后出现的变化。
  ```



#### handler.setPrototypeOf()

​		**`handler.setPrototypeOf()`** 方法主要用来拦截 [`Object.setPrototypeOf()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf).

​		**Object.setPrototypeOf()** 方法设置一个指定的对象的原型 ( 即, 内部[[Prototype]]属性）到另一个对象或  [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null)。

