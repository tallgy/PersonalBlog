---
title: JavaScript-ES6
date: 2022-03-05 21:53:11
tags:
 - JavaScript
 - 随笔
categories:
 - JavaScript
 - 随笔
---



# ES6

## 介绍

​		ECMAScript，简单来说就是发布规定脚本语言的标准。

​		ES6，全称就是 ECMAScript 6.0，是在 2015 年正式通过的。



## 内容



### let const

​		新建了 let 和 const 声明

​		let const 和 var 的区别：

* 作用域： let const 的 作用域 处于括号级作用域，var 是一个函数级的作用域，关于作用域的问题，我们后面在进行详细的学习
* 变量提升：let ，const 不会出现变量提升。而var会出现变量提升的效果。
* const 常量，不可变。



### 模板字符串

​		\` \`，ES6增加了模板字符串，可以进行换行，以及可以在字符串中拼接表达式 ${}。



### 箭头函数

​		添加了 箭头函数，() => {}

* 函数上下文：this 的指向不同，普通函数的this指向处于调用时的调用栈的this指向。而箭头函数的函数上下文在定义时已经确定了，无法改变。
* 不存在构造器函数：因为不存在构造器函数，所以不能使用new方法，会报错。



### 函数参数默认值

​		



### Spread/Rest 操作符

​		简单来说就是 ... ，我们可以使用 ... 将数组解开为单个元素，也能将元素合并为一个数组。



### 二进制和八进制的字面量

* 0o xx：八进制
* 0b xx：二进制



### 解构赋值

​		可以将对象和数组进行解构

```
let {a} = obj
let [c, d] = arr
```

​		注意：

* 对象的解构，是针对名字进行的，没有的键会为undefined，我们也可以赋予默认值
* 数组的解构是通过下标的对应，对于不要的我们可以使用 ,, 将其赋值给不存在的变量，也可以使用 spread 和 rest 操作符进行合并赋值。



### 类

​		ES6 新增了一个关于 Class 的语法糖。

​		类的ES5的实现

​		new 方法的过程：

* 创建一个this的空对象，同时继承函数的原型。
* 然后再执行方法。
* 如果返回的不是对象，那么就返回this。否则返回对象。

```
一次new操作：
创建一个this，原型指向了fn的原型
let this = Object.create(fn.prototype)

调用fn
let res = fn.apply(this, args)

通过判断res的类型来判断返回是res还是this
return typeof res === 'object' ? res : this;
```



构造器

​		ES6的构造器，就是类在进行new方法时会执行的一个方法。对于没有构造器的方法不能使用new方法。

​		类的构造器，实际上就是在new方法执行时会执行的函数

​		同时使用 class 创建的函数，是一个 不可枚举的。存在于实例的 proto 中。 fn.proto === Fn.prototype 。



super

​		构造器内部可以调用super方法执行父类的构造器。

* 在普通方法中，super指向了父类的原型，this是子类的实例

* 在静态方法中，super指向了父类，this就是子类
* 同时，不能直接输出super，super需要以方法或属性的形式调用 super() super.fn() super.cc



继承

​		class A extend B

* 继承可以使用super来调用父类的方法和构造器
* 所谓的继承就是，可以将父类的方法放到子类进行调用，同时父类的构造器生成的属性也会放到子类。
* 所以我们又知道，子类的实例的属性是子类的构造器的执行的结果。子类实例的原型就是子类的原型。子类原型的proto就是继承的类的原型。

继承的ES5的实现

我们需要注意的点：

* 需要执行父类的构造器方法
* 需要将父类的原型设置为自身的原型，一般来说，原型是不会赋予属性的，所以一般默认都是一个空的对象，然后这个对象的proto属性是Object.prototype。

```
function Child() {
	// 调用 父函数的方法，使用call方法进行调用，
	// 这里思考的话需要考虑，如果父函数存在对象返回值的情况，那么会返回的是对象，而不是call方法了。这个需要进行解决。
	Parent.call(this, args)
	
	// 这里写自己的方法
	...
}

实现原型的继承

Parent.prototype

```

```
class Parent {
  constructor() {
    this.p = 1

    return {
      p: 2,
    }
  }
}
class Child extends Parent {
  constructor() {
    super()
    this.c = 3
  }
}

我们执行之后，发现，返回值 p=2，而不是1.
这里我刚刚百度了一下，在继承里面，ES6，是先super，然后再this，所以对于继承里面，如果没用super，将不能使用this。所以我们大概知道就是 super，先像new一样，然后通过返回将其加入this，然后再使用。
```

```
简单的一个实现 Super
使用：Super.call(this, Parent)

function Super(Parent) {
  const ctx = {}
  const res = Parent.call(ctx)
  const r = typeof res === 'object' ? res : ctx

  Object.assign(this, r)
}
```







