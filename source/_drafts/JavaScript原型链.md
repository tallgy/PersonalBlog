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

​		我们要这样思考，这个new方法。

* new 一个方法，会创建的一个this的指向。
* 然后会执行这个方法，执行结束之后。
* 创建一个 \_\_proto\_\_  指向了 方法的 prototype 的指向。
* 然后就进行返回。



​		**JavaScript 继承**：

继承，简单的理解就是，子类的实例可以访问父类的方法

同时我们可以知道，父类的方法，创建的实例才能进行调用。

而父类的prototype，已经可以算作为父类的父类的方法了。

所以，我们在不考虑上级的方法之前，需要将父类的方法进行传递，

这里，可以考虑使用一个新的对象进行指定。