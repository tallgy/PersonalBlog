---
layout: draft
title: js遍历对象的方法
date: 2022-01-19 17:32:25
tags:
 - JavaScript
 - 随笔
categories:
 - JavaScript
 - 随笔
---



#  JavaScript 遍历对象的方法

```
Object.keys
forin
Object.getOwnPropertyNames
```



## Object.keys

使用 Object.keys，会返回一个数组，数组是一个由键构成的

元素均为对象自身的，可枚举的属性

​	代表了，不会查找原型链，以及需要可枚举

```
const obj = {
	a: 1
}
const pro = {
	b: 2
}
obj.__proto__ = pro

Object.keys(obj).log
	a
```



## forin

forin循环，获取的是key值，会向原型链进行获取。但是不会查找不可枚举的属性

​	可以使用 hasOwnProperty 来判断该属性是否是属于自身的。同时对于不可枚举的也是可以进行判断的。

```
const obj = {
	a: 1
}
const pro = {
	b: 2
}
obj.__proto__ = pro
Object.defineProperty(obj, 'c', {value: 3, enumerable: false}) // 不可枚举属性

for key in obj
	log.key
	log.obj[key]
	log.obj.hasOwnProperty(key)
  a, b
```



## Object.getOwnPropertyNames

返回自身的属性数组，不包括原型链

但是包括不可枚举属性

同时对于一个数组，他的 getOwnPropertyNames 有length，说明了这个length是存在于自身的

```
const obj = {
	a: 1
}
const pro = {
	b: 2
}
obj.__proto__ = pro
Object.defineProperty(obj, 'c', {value: 3, eumerable: false})	// 添加一个不可枚举属性

Object.getOwPropertyNames(obj)
	a, c
```



## 题外话：forof

forof 其实在前面就已经说过了，这个是一个根据迭代器进行的循环

面试也问过，为什么forin可以遍历数组，当时我的理解是 forin 实际上并不是遍历数组，他是遍历一个对象，现在我也是这样认为的，我认为forin 不是遍历数组，他内部的遍历实际是对象，只是因为js的任何都是对象的原因，所以 forin 遍历数组获取到的下标，这样一看，他不是很像是对象的键吗，数组只是存在了一个length方法。而对象没有而已。

我认为forof才是JavaScript对于遍历数组而创建一个方式，因为forof使用的迭代器，是数组才有的，创建的对象是没有的，当然这个我们也能进行自己编写，具体的可以看看前面写的 迭代器部分。

