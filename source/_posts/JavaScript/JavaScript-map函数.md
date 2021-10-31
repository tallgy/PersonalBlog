---
title: JavaScript-map函数
date: 2021-10-25 16:48:23
tags:
 - JavaScript
 - Array
 - map函数
 - 源码
categories:
 - JavaScript
 - Array
---



#  JavaScript之Array.map函数



## 使用方法

```
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map
```

`**map()**` 方法创建一个新数组，其结果是该数组中的每个元素是调用一次提供的函数后的返回值。



**参数**

​	**回调函数，callback**

​		参数分别为，value，index，array，第二个和第三个参数是可选的。

```
const array1 = [1, 4, 9, 16];

// pass a function to map
const map1 = array1.map(x => x * 2);

console.log(map1);
// expected output: Array [2, 8, 18, 32]
```

​	**thisArg**

​		在执行callabck函数时，被用作 `this`

## 简单的源码实现

通过上面的参数描述，我们可以知道，

我们需要先对数组进行循环，然后将其带入callback，返回值会push入一个res数组，

然后因为第二个参数是this的指向，所以我们需要在判断是否携带了第二个参数**(  const _this = thisArg || this;)**

将其this的指向进行改变，使用了call方法。

```
function myMap(callback, thisArg) {
  const result = [];
  const arr = this;
  const _this = thisArg || this;

  for (let i = 0; i < arr.length; i++) {
    result.push(callback.call(_this, arr[i], i, arr));
  }

  return result;
}
```



## 一些面试时遇到的问题

```
const arr = [1, 2, 3, 10, 111];

arr.map(parseInt)

这个可能有的没有理解的话会有点懵逼
其实理解了就很简单了。
这就是把parseInt作为一个回调函数传递给了map，
map的回调函数的参数是，value， index， arr
而parseInt，可以带两个参数，这两个参数，一个是要转化的字符串（数字也可以），一个是要转化的进制。

所以我们就能够理解了，
对于 arr 1， 2， 3， 10， 111
会分别传递
	1， 0
	2， 1
	3， 2
	10， 3
	111， 4
其中，0代表了默认转换，对于字符串没有进制前缀的，就是默认十进制，具体的转换规则后续写 parseInt 的时候再说明。

我们可以看出，1对应的默认进制转换，2对应的1进制，3对应的2进制，10对应的3进制，111对应的4进制，因为parseInt的第二个参数范围是2-36，所以对于其他的除0以外直接NaN。
所以对应的结果是 1, NaN, NaN, 3, 21.
```

对于其他的就也可以按照这个思路进行思考了。