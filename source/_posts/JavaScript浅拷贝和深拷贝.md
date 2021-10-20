---
title: JavaScript浅拷贝和深拷贝
date: 2021-10-19 15:28:12
tags:
 - JavaScript
 - 浅拷贝
 - 深拷贝
---



# 浅拷贝



很简单，就是只拷贝一层，不拷贝多层，因为对于引用数据类型，

他的数据是存放在堆中的，我们只是在栈中存放了他的地址。所以浅拷贝的问题就是对于引用数据类型还是没有进行拷贝。

```
一个简单的demo

这里，有两个循环操作。对于forin，他会循着原型链向上查找，
而对于 Object.keys() 和 Object.getOwnPropertyNames()，他只会返回属于自身的属性。
而不会向上寻找原型链
```

```
let obj1 = {
  a: 1
};

let obj2 = {
  b: 2
};
obj2.__proto__ = obj1;

for (const obj2Key in obj2) {
  console.log(obj2Key);
}
console.log(Object.keys(obj2))
console.log(Object.getOwnPropertyNames(obj2))
```

```
b
a

[ 'b' ]

[ 'b' ]
```



所以我们考虑到是否要将原型链上的一起拷贝之后，再进行拷贝

```
keys代表了要拷贝的key

const obj = {}

for (const key in keys) {
  obj[keys[key]] = obj2[keys[key]];
}
```

这个就是浅拷贝了。比较简单



## 同时也可以使用 `Object.assign()` 进行浅拷贝

```
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
```

```
Object.assign(target, source);

第一个参数是目标对象, 第二个参数是源对象,
我们会将源对象的值拷贝给目标对象,对于重复的会被覆盖.
然后再将目标对象返回.
```

```
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5, x: {
    a: 1
  } };

const returnedTarget = Object.assign(target, source);

console.log(target);
// expected output: Object { a: 1, b: 4, c: 5, x: { a: 1 } }
console.log(returnedTarget);
// expected output: Object { a: 1, b: 4, c: 5, x: { a: 1 } }

source.x.a = 3;
console.log(target);
// expected output: Object { a: 1, b: 4, c: 5, x: { a: 3 } }
console.log(returnedTarget);
// expected output: Object { a: 1, b: 4, c: 5, x: { a: 3 } }
```



# 深拷贝

和浅拷贝不同的是，深拷贝，是要进行递归，吧对象里面的对象也要拷贝出来。



## 第一种：使用 JSON的方法，这个方法我也觉得很巧妙，

除了不能拷贝 function，正则，Symbol之外，其他都很不错。

```
方法也很简单

先对其进行string化，然后再json化，所以简单的使用没有问题。
JSON.parse(JSON.stringify(obj))
```



```
let obj = {
  reg : /^aaa$/,
  fun: function(){ console.log(1) },
  syb:Symbol('foo'),
  aa:'aa'
};
let cp = JSON.parse(JSON.stringify(obj));
console.log(cp);
```

```
{ reg: {}, aa: 'aa' }
```



## 第二种，就是简单的类型判断加递归了

就判断是否为基本数据类型，如果是那么就进行复制，如果不是那么就进入递归

其中，对于正则，基本数据类型，可以直接赋值。

对于对象，使用递归，

对于函数，我这里下面再讲



#### 类型判断

这里，先用`typeof`判断是否为 `object`，

然后，对于object，存在 null，对象，数组，正则和使用new进行创建的基本数据类型

​	在这里，我把使用new方法创建的基本数据类型也算为基本数据类型。

对于非object，有function，和基本数据类型。

```
function getType(target) {

  const type = typeof target;
  if (type == 'object') {
    if (target === null) {
      // 空
      return 'null';
    } else if (Array.isArray(target)) {
      // 数组
      return 'array';
    } else if (target.constructor === RegExp) {
      //正则
      return 'regexp'
    } else {
      // 对象，这里还要判断是否为map和set对象。
      //并且，map，set只能使用 forof 进行循环，或者使用自带的 oreach 进行循环。
      if (typeof target.valueOf() === 'object') {
        return 'object';
      } else {
        return 'basic';
      }
    }
  } else if (type == 'function') {
    return 'function';
  } else {
    return 'basic';
  }
}
```

```
这里也可以使用 Object.prototype.toString.call(); 方法进行调用来判断是什么类型的.这个就比较方便,直接可以将map,set,对象,数组,基本数据类型等都辨别出来.
可以看看我写的另一个关于 类型判断的blog ,里面有.
```



#### 然后就是递归

```
function deepClone(target) {
  let result;

  for (const key in target) {
    result[key] = recursion(target[key]);
  }

  return result;
  
  function recursion(target) {
    if ('基本数据类型') {
      return target;
    } else if ('函数') {
		
    } else {
      //对象
      return recursion(target)
    }
  }
}
```



#### 对于函数如何进行拷贝

前几天面试问到了..

我的想法，有三种方式，

​	**第一种：使用toString，然后再使用eval。**	

```
let fn = function () {
  console.log(1);
}

function fn1() {
  console.log(1)
}

这里我对于eval函数的使用还是不够了解。
这里，我不知道为什么， eval(fn.toString()) 没有返回值，所以我写了一个箭头函数字符串进行包装和执行。
	这里，需要使用一个箭头函数进行包装，如果使用普通函数进行包装。也没有用处。
let fn2 = eval('() => ' + fn1.toString())(); //这里，fn和fn1都可以运行。


这样写也可以。
eval(`
  (() => fn)()
`)()
```

```
应该是因为需要使用()进行包裹.
let a = eval(`
  (function fn() {
    console.log(1);
    return 1;
  })
`);
a()
console.log(a.toString())
```

```
1
function fn() {
    console.log(1);
    return 1;
  }
```

​	

**第二种：使用new Function('return ' + func.toString())();**

```
let f = new Function('return ' + fn.toString())()

f();
```

​	**第三种：使用bind函数进行一次绑定。但是这个对于有传参的函数会有所不同。**

```
这个有个要求，因为bind函数实则就是对于一个函数进行了call方法，但是是一个延迟的调用。

从一个简单的逻辑来说，应该是并没有进行一个从底层复制操作的。因为是直接进行的 简单赋值。

let f = fn.bind()
f();
```

