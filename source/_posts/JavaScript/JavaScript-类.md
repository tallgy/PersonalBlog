---
title: JavaScript-类
date: 2022-02-15 11:18:08
tags:
 - JavaScript
 - Class
categories:
 - JavaScript
 - Class
---



#  JavaScript 类

## 方法

### 普通方法

​		普通方法将会给继承给实例。同时方法是存在于实例的 **\_\_proto\_\_** 里面的，同时是处于不可枚举的状态。

```
class c extends p {
	fn() {
	
	}
}

let a = new c()
a.fn()
```

### 静态方法

​		静态方法是一个可以直接进行类名调用的方法。对象实例不能调用。静态方法可以继承。

```
class c extends p {
	static fn() {
	
	}
}

c.fn()
```

## extends

```
class Children extends Parent {
	fn() {}
}
```

​		简单来说就是继承，和Java差不多。创建一个Children类，然后继承了Parent，然后后面写的方法就是Children自身的方法。



## super

​		super 关键字，既可以当作函数使用，也可以当作对象使用。

### 当作函数使用

​		子类的构造函数必须要执行一次 super() 函数。用于调用父类的构造函数，并且只能在构造函数内调用。

```
class children extends parent {
	constructor(name) {
		super(name)
		...
	}
}
```

### 当作对象使用

​		super 作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。







