---
title: if in 用法
date: 2022-02-12 16:09:34
tags:
 - 随笔
 - JavaScript
categories:
 - JavaScript
 - 随笔
---



#  if in 的用法

```
const obj = {
  a: 1,
  b: 2,
}

obj.__proto__.c = 3

Object.defineProperty(obj, 'd', {
  value: 123,
  enumerable: false
})

if ('a' in obj) {
  console.log('a')
}
if ('c' in obj) {
  console.log('c')
}
if ('d' in obj) {
  console.log('d')
}
```

```
a
c
d
```



所以我们可以知道，if ( in )

会把 自身的，原型链上的。以及不可枚举的都会显示。