---
title: eval函数
date: 2021-10-20 11:01:49
tags:
 - JavaScript
 - eval
categories:
 - JavaScript
 - Global_Objects
---



#  eval函数

**文档**

```
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/eval
```



这里学习使用eval函数的原因是, 前几天遇到一个问题是拷贝函数, 

所以我就想说,先使用tostring, 然后再使用eval转为函数返回进行使用, 这样就进行了一次函数的复制.



```
let f = eval(`
function fn() {
  console.log(1);
  return 2;
}
`);

console.log(f);
```

```
你会发现结果 是不如人意的?我的想法就是这个字符串方法会被转义为一个函数赋值给f.但是却什么都没有,函数没有执行,并且没有返回值.

undefined
```

然后转念一想.可能是因为虽然转义为函数了,但是没有执行和返回.他只是定义了一下函数而已.

```
let f = eval(`
function fn () {
  console.log(1);
  return 2;
};
`);

fn();
```

```
结果真的有输出.所以这样一想.我可以在内部加上一个立即执行函数啊
```

```
function fn () {
  console.log(1);
  return 2;
};

//我这里只是定义立即执行函数,但是没有执行.只需要使用 f(), 便可以进行函数调用了.
let f = eval(`
  (${fn.toString()})
`);

console.log(f.toString());
console.log(f());
```

```
执行结果:
function fn () {
  console.log(1);
  return 2;
}
1
2
```

