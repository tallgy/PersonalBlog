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
(function() {
	function Fn() {}
	Fn.prototype = Parent.prototype
	Child.prototype = new Fn()
	Child.prototype.constructor = Child
})()

因为一个new方法就是要，创建一个原型指向了父类的原型的一个对象，然后执行构造器函数
区别就是使用了new方法，然后中间会创建一个对象，对象的原型指向了父类原型，然后我们使用constructor修改了对象的构造器指向，就不会修改到父类的构造器了。

Child.prototype = Parent.prototype
// 这里是为了修改 构造器的指向，但是因为我们是直接复制的原型，所以这里的修改会影响到父类的构造器的指向。
Child.prototype.constructor = Child
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
思路：就是先创建一个空对象，进行调用，然后通过返回值类型来判断，哪个应该合并为this
	缺点：因为是使用对象，而并不是真正的上下文this，所以不能使用一些需要依靠this操作的行为。以及赋值给this是依靠的 Object.assign 进行的操作。

function Super(Parent) {
  const ctx = {}
  const res = Parent.call(ctx)
  const r = typeof res === 'object' ? res : ctx

  Object.assign(this, r)
}
```



### Promise

​		简单来说就是用来传递异步的消息。

​		在以往，我们使用异步的形式都是使用的回调函数，就是将回调函数作为参数传递过去，然后方法那边的在异步结束之后再调用这个函数。

​		同时Promise存在三个状态：

* pending：初始状态
* fulfilled：成功的状态
* rejected：失败的状态

​	**注意**：当状态从pending进入另外两个状态是，将不能改变了。

​		只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

![promises](JavaScript-ES6/promises.png)



#### Promise.then():

​		then() 方法返回一个 Promise (en-US)。它最多需要有两个参数：Promise 的成功和失败情况的回调函数。

​		两个参数：代表了 res 和 err（可选）

​		如果是Promise.then()，如果后面再有 .then() 但是却没有 return一个Promise的，那么就会进入 resolve(undefined) 的情况

返回值：

分为：返回一个值，没有返回，接受状态，拒绝状态，和未定状态。

- 返回了一个值，那么 `then` 返回的 Promise 将会成为接受状态，并且将返回的值作为接受状态的回调函数的参数值。
- 没有返回任何值，那么 `then` 返回的 Promise 将会成为接受状态，并且该接受状态的回调函数的参数值为 `undefined`。
- 抛出一个错误，那么 `then` 返回的 Promise 将会成为拒绝状态，并且将抛出的错误作为拒绝状态的回调函数的参数值。
- 返回一个已经是接受状态的 Promise，那么 `then` 返回的 Promise 也会成为接受状态，并且将那个 Promise 的接受状态的回调函数的参数值作为该被返回的Promise的接受状态回调函数的参数值。
- 返回一个已经是拒绝状态的 Promise，那么 `then` 返回的 Promise 也会成为拒绝状态，并且将那个 Promise 的拒绝状态的回调函数的参数值作为该被返回的Promise的拒绝状态回调函数的参数值。
- 返回一个未定状态（`pending`）的 Promise，那么 `then` 返回 Promise 的状态也是未定的，并且它的终态与那个 Promise 的终态相同；同时，它变为终态时调用的回调函数参数与那个 Promise 变为终态时的回调函数的参数是相同的。



#### Promise.catch():

​		catch() 方法返回一个Promise (en-US)，并且处理拒绝的情况。它的行为与调用Promise.prototype.then(undefined, onRejected) 相同。 (事实上, calling obj.catch(onRejected) 内部calls obj.then(undefined, onRejected)).

​		参数：onRejected

返回值：
		一个Promise.

​		默认一个 resolve，可以返回rejected



#### Promise.finally():

​		finally() 方法返回一个Promise。在promise结束时，无论结果是fulfilled或者是rejected，都会执行指定的回调函数。这为在Promise是否成功完成后都需要执行的代码提供了一种方式。这避免了同样的语句需要在then()和catch()中各写一次的情况。

​		参数：onFinally



#### Promise.all:

​		Promise.all() 方法接收一个promise的iterable类型（注：Array，Map，Set都属于ES6的iterable类型）的输入，并且只返回一个Promise实例， 那个输入的所有promise的resolve回调的结果是一个数组。

​		接受一个 promise的iterable类型

​		就是可迭代的promise。需要全部返回resolve才能resolve

​		只要任何一个输入的promise的reject回调执行或者输入不合法的promise就会立即抛出错误，并且reject的是第一个抛出的错误信息。



#### Promise.allSettled:



#### Promise.any:

​		Promise.any()` 接收一个 Promise 可迭代对象，只要其中的一个 `promise` 成功，就返回那个已经成功的 `promise



#### Promise.race:

​		将与第一个传递的promise相同的完成方式被完成。就是说，在一个可迭代对象里面，我们将获取到所返回的第一个结果，不管是resolve和rejected。



### generator 生成器

​		生成器对象是由一个 generator function 返回的,并且它符合可迭代协议和迭代器协议。

​		实例：

```
一个无限的迭代器

function* idMaker(){
    let index = 0;
    while(true)
        yield index++;
}

let gen = idMaker(); // "Generator { }"

console.log(gen.next().value);
// 0
console.log(gen.next().value);
// 1
console.log(gen.next().value);
// 2
// ...
```

​		通过这个我们可以发现一个可迭代协议和可迭代对象，可迭代协议是一个Es6的补充规范。

​		简单来说，一个可迭代对象就需要实现这个可迭代协议，会实现一个 @@iterator 方法。（原型链上存在也可以），在for of 的循环和使用 spread和rest操作符的时候就会使用上这个可迭代协议。

​		可迭代协议的方法就是返回一个next方法。方法返回一个 done 和 value 属性。

​		而这个就是我们生成器的返回的特点，简单来说就是我们生成器就是一个满足可迭代协议而存在的一个方法。



### 模块化

​		模块化简单分为 CommonJS、AMD、CMD和现在的ES6

​		其中 AMD 和 CMD 都是参照的 CommonJS

CommonJS：

​		NodeJs发布的。

​		require引入、exports导出

AMD、CMD:

​		define 定义模块，require实现加载。

ES6：

​		使用 import 导入

​		export 导出

​		关键字有import，export，default，as，from。



ES6 和 CommonJS的区别：

* ES6 是浅拷贝，修改会影响原理的值。
  * CommonJS的深拷贝
* CommonJS是运行时加载。
  * ES6是编译时输出接口，可以单独加载接口。

