---
title: 数组的emptyItems
date: 2021-11-13 21:45:21
tags:
 - JavaScript
 - Array
 - 随笔
categories:
 - JavaScript
 - 随笔
---



#  数组的empty-items

## empty的产生

​		首先，先说一下如何产生empty

* 使用 new Array函数进行创建，并没有初始化

```
let a = new Array(10);
console.log(a);
```

* 使用 length方法重新定义了长度，长度比原来的数组长，那么会导致多余的变为 empty-items。

```
let a = [1];
a.length = 10;
```



## empty的问题

​		开始我也没有在意这个empty的问题，但是后来发现对于empty-items 使用一些方法是不会进行循环的。这就造成了很大的问题。

* map等数组的高阶函数，
* forin循环

```
let a = new Array(10);

a.map((v) => {
  console.log(v);
})

for (const key in a) {
  console.log(a[key])
}
```

​		但是forof循环是可以的，fori循环也可以，输出的值就是undefined。



## 原因

当然具体的原因我也不知道，我们可以这样理解，JavaScript的数组也是一个特殊的对象形式，所以对于这些empty-items，其实并没有真正的存在，就类似于一种离散的数组形式，所以为什么循环不到，因为对于不存在的是无法循环的。但是为什么forof是可以循环的呢，我们也许可以这样思考，forof是使用的迭代器循环，所以并不是循环到了，只是这是一个数组的迭代器的形式，所以这样才成功的循环了。

