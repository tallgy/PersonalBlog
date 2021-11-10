---
title: JavaScript原型链
date: 2021-11-09 20:57:32
tags:
 - JavaScript
 - 原型链
 - 随笔
categories:
 - JavaScript
 - 随笔
---



 **注意：**

**这里的内容为本人的理解，不敢保正。**



#  JavaScript原型链









# JavaScript 继承



```
function Fn() {
  this.name = 1;
}

let a = new Fn();

此时的
a.__proto__ = Fn.prototype
a.__proto__.constructor = Fn;
```

## new 方法的行为

​		我们要这样思考，这个new方法。

* new 一个方法，会创建的一个this的指向。
* 然后会执行这个方法，执行结束之后。
* 创建一个 \_\_proto\_\_  指向了 方法的 prototype 的指向。
* 然后就进行返回。

```
let _this = {};
let result = Fn.apply(_this, args);
_this.__proto__ = Fn.prototype;
return typeof result === 'object' ? result : __this__;
```

​		同时这里，在进行 new 方法的调用时，我们知道，如果返回的是一个引用类型，那么便会直接返回这个引用类型，如果返回的是一个值类型，那么返回的便是自己所创建的this指向。我们可以知道一个引用类型，无论你是使用的new方法，还是直接进行的创建的引用类型。其结果都会创建原型。

**注意**：

​		对于一个值类型，进行了new 方法，也会变成引用类型的。

```
let x = new Number(1);
let y = new Number(1);

console.log(x === y); //false
```



## **JavaScript 继承**：

### 继承的思路

继承，简单的理解就是，子类的实例可以访问父类的方法

同时我们可以知道，父类的方法，创建的实例才能进行调用。

而父类的prototype，已经可以算作为父类的父类的方法了。

所以，我们在不考虑上级的方法之前，需要将父类的方法进行传递，

这里，可以考虑使用一个新的对象进行指定。如果我一个新对象



### 原型和构造器

这里和Java的一个不同点

我印象中，Java是子类有多个，父类越来越少的一种情况，

<img src="JavaScript原型链/image-20211110100356010.png" alt="image-20211110100356010" style="zoom:67%;" />

​		一开始，我是这样思考的，构造函数和实例是对应的，而构造函数的原型应该是属于上层的一个实例了。

​		但是你仔细观察，发现，一个构造函数的原型是和构造函数进行了对应。思想应该是每个实例都有一个原型，但是其原型默认是没有constructor的（这里可以使用 hasOwnProperty 进行证明）。我们所使用的constructor属性其实都是我们的实例的原型  \_\_proto\_\_ 中的constructor，但是实例的原型为什么会带有constructor，这里我的认为是，一个函数默认在词法编译阶段，会默认指向一个prototype，而这个行为就是 

```
Fn.protorype = new Object();
Fn.prototype.constructor = Fn;
```

​	简单来说，就是在没有指定的情况下，默认先创建一个顶级的对象实例，里面只有最原始的方法，然后再对这个实例创建一个本函数的构造行为。

<img src="JavaScript原型链/image-20211110101346723.png" alt="image-20211110101346723" style="zoom:67%;" />

​		而这个JavaScript是每个子类，都会将父类进行一次实例的操作，所以造成了每个父类的实例和子类的构造函数一一对应了。



### new 方法实现

#### **`new` 关键字会进行如下的操作：**

1. 创建一个空的简单JavaScript对象（即`{}`）；
2. 为步骤1新创建的对象添加属性**__proto__**，将该属性链接至构造函数的原型对象 ；
3. 将步骤1新创建的对象作为`this`的上下文 ；
4. 如果该函数没有返回对象，则返回`this`。

#### **当代码 `new Foo(...)` 执行时，会发生以下事情：**

1. 一个继承自 `Foo.prototype` 的新对象被创建。
2. 使用指定的参数调用构造函数 *`Foo`*，并将 `this` 绑定到新创建的对象。`new Foo` 等同于 *`new Foo`*`()`，也就是没有指定参数列表，*`Foo`* 不带任何参数调用的情况。
3. 由构造函数返回的对象就是 `new` 表达式的结果。如果构造函数没有显式返回一个对象，则使用步骤1创建的对象。（一般情况下，构造函数不返回值，但是用户可以选择主动返回对象，来覆盖正常的对象创建步骤）

```
function create(Fn, ...args) {
  let _this = {};
  let result = Fn.apply(_this, args);
  _this.__proto__ = Fn.prototype;
  return typeof result === 'object' ? result : _this;
}
```



### 继承的方式

#### 原型链继承

```
Children.prototype = new Father();
Children.prototype.constructor = Children;
```

​		很简单的理解，就是直接将子类的原型修改为了父类的实例，然后再对实例的constructor进行一个校准，修改为子类的构造。

​		执行new方法的行为的步骤：

* 产生一个this指向，\_\_proto\_\_  的添加，指向为children.prototype。就是 Father的实例。
* 执行children的构造函数，this指向的调用。
* 返回。



**缺点**：

* 对于要在原型上添加修改的方法，需要在 new Father 之后，因为这个是直接将一个地址进行了转换。
* 其次就是不会将参数传递到父类。只适合不带参数的。



#### 组合继承

```
function Children() {
	Father.call(this);
}
Children.prototype = new Father();
Children.prototype.constructor = Children;
```

​		这个和原型链继承大同小异，就是多了一个在构造函数内部使用call方法调用父类。使用call方法调用时是不会产生 \_\_proto\_\_  和 constructor 的。但是你可能会发现自己测试还是有啊，但是你仔细观察，其实你自己测试所生成的 \_\_proto\_\_  和 constructor 是在创建对象的时候生成的。所以，其实并没有真正的生成。



**缺点**：

* new 方法 和 call 方法都生成了一份实例。



#### 寄生组合继承

```
function Children() {
	Father.call(this);
}
(function () {
	let Super = function() {};
	Super.prototype = Father.prototype;
	Children.prototype = new Super();
	Children.prototype.constructor = Children;
})();
```

​		我们可以查看这个和 组合继承的区别，使用了一个空方法将Father的原型复制了过来，所以new所生成的方法就只生成了一个空对象，和一个原型。Father的方法实则已经在call方法里面进行了调用。





