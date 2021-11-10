---
title: JavaScript箭头函数
date: 2021-11-10 19:42:56
tags:
 - JavaScript
 - 随笔
 - 箭头函数
categories:
 - JavaScript
 - 随笔
---



#  JavaScript箭头函数

```
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions
```



​		**箭头函数表达式**的语法比[函数表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/function)更简洁，并且没有自己的`this`，`arguments`，`super`或`new.target`。箭头函数表达式更适用于那些本来需要匿名函数的地方，并且它不能用作构造函数。

## 特点

* 没有自己的this，它只会从自己的作用域链的上一层继承this。
* 没有arguments，不能使用自己的arguments对象。使用的arguments都是自己作用域的arguments。
* 不能用做构造函数，代表了不能使用new方法。使用了new方法会抛出异常。
* 与严格模式的关系
* 通过call方法和apply进行调用的this指向被忽略
* 没有prototype属性
* 不能使用yield关键字，不能用作函数生成器
* 参数和箭头不能换行
* 箭头函数具有与常规函数不同的特殊[运算符优先级](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)解析规则。



## 语法

```
let fn = (args) => {
  xxxx
  return ;
}
```

​		对于单个参数的可以简写，()

```
v => {}
```

​		对于只有一条return 语句的，{}

```
() => xxx;
```



**高级语法**：

​		

```
//加括号的函数体返回对象字面量表达式：
params => ({foo: bar})
```

​		剩余参数，简单来说就是一个es6的一个赋值方式 ...args，后面的参数都会放入这个数组

​		默认参数，就是不带参数和带的undefined的参数 param=defaultValue

```
//支持剩余参数和默认参数
(param1, param2, ...rest) => { statements }
(param1 = defaultValue1, param2, …, paramN = defaultValueN) => {
statements }

```

​		就是es6的解构方法。

```
//同样支持参数列表解构
let f = ([a, b] = [1, 2], {x: c} = {x: a + b}) => a + b + c;
f();  // 6
```



## this指向

​		箭头函数的一个特点就是this指向的固定，这里我们可以知道箭头函数的this指向是定义时词法作用域的指向。

​		这里可能就有一个思考了，为什么这个会和定义时的词法作用域相关。而不是和使用时的作用域相关。我们也许就该这样思考了，**也许**说明在定义阶段，如果有使用了this的方法，会进行一个提前的指向。



​		**这里有个小知识点**：

* 就是node端和浏览器端，对于var的定义不同：
  * node不会将var定义的变量加入全局。
  * 浏览器端会将var定义的变量加入全局。



