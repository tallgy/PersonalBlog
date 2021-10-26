---
title: JavaScript-parseInt函数
date: 2021-10-25 19:41:27
tags:
 - JavaScript
 - parseInt
categories:
 - JavaScript
 - Global_Objects
---



#  parseInt函数

```
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt
```

​	**parseInt(\*string\*, \*radix\*)**  解析一个字符串并返回指定基数的十进制整数， `radix` 是2-36之间的整数，表示被解析字符串的基数。



## 参数解析

**string**

​		要被解析的值。如果参数不是一个字符串，则将其转换为字符串(使用  `ToString `抽象操作)。字符串开头的空白符将会被忽略。

**radix**

​		从 `2` 到 `36`，表示字符串的基数。例如指定 16 表示被解析值是十六进制数。请注意，10不是默认值！简单来说就是设置进制，一个2~36的进制



## 返回值

一个整数，

或者NaN

​	转换NaN的情况

​		`radix` 小于 `2` 或大于 `36` ，或

​		第一个非空格字符不能转换为数字。



**注意：**

​	这个在进行进制运算时，不是先转为十进制在运算，而是直接进行对应的进制转换，

​		比如：	33 ， 2 此时是：33转为2进制的转换，因为2进制每位数最大值是1，所以33直接NaN，但是如果是12，就会转化为1.



## 面试问到的：

```
const arr = [1, 2, 3, 10, 111];
arr.map(parseInt)
```

```
简单的解析一下，
首先map内部的回调函数的三个参数分别为 value，index，array
然后parseInt会使用两个参数，一个是要转换的字符串，一个是进制
所以 按照顺序就会变成： 1,0	2,1		3,2		10,3	111,4
	对于0，会使用默认形式，对于1，会直接NaN，对于后面的就按照正常进制的运算
	所以结果是： 1,NaN,NaN,3,21
```

