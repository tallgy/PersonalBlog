---
title: JavaScript-ES6解构
date: 2021-11-11 22:14:32
tags:
 - JavaScript
 - ES6
 - 解构赋值
categories:
 - JavaScript
 - ES6
---



#  JavaScript 解构赋值

```
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
```



​		**解构赋值**语法是一种 Javascript 表达式。通过**解构赋值,** 可以将属性/值从对象/数组中取出,赋值给其他变量。

​		简单来说，就是对于一个赋值变得简单起来。



## 语法

### [a, b] = arr;

* 多余的不会被赋值
* 少的会变成undefined
* 注意要避免arr为null和undefined，对象同样，需要是数组和对象才行。

```
let arr = [1, 3, 5];

let [a, b] = arr;
console.log(a, b);

1, 3
```



### [a, b, ...rest] = arr;

* 这里，对于多余的都会变成第三个的数组
* 少于的，前面两个是undefined， 第三个数组会是一个空数组。
* 剩余元素必须在最后一个。

```
let arr = [1, 3, 4, 2, ];

let [a, b, ...res] = arr;
console.log(a, b, res);

1 3 [ 4, 2 ]
```



### ({ a, b } = obj);

* 这个是以名称为主，需要找到obj内部的a，如果没有，那么就会变成undefined，其次，这里的值是可以是原型上的。 obj.\_\_proto\_\_ 只要原型链上存在也会赋值。
* 其次，这里需要添加括号，或者是定义时进行解构赋值，这里的作用可能是避免大括号的影响了编译的判断。
* **注意**：赋值语句周围的圆括号 `( ... )` 在使用对象字面量无声明解构赋值时是必须的。 `{a, b} = {a: 1, b: 2}` 不是有效的独立语法，因为左边的 `{a, b}` 被认为是一个块而不是对象字面量。

* 然而，`({a, b} = {a: 1, b: 2})` 是有效的，正如 `var {a, b} = {a: 1, b: 2}`你的 `( ... )` 表达式之前需要有一个分号，否则它可能会被当成上一行中的函数执行。

```
let obj = {
  a: 10,
  b: 20,
  c: 30
};
let a,c;
({a, c} = obj);
console.log(a, c);

10, 30
```

​		or

```
let obj = {
  a: 10,
  b: 20,
  c: 30
};
let {a, b} = obj;
console.log(a, b);

10, 20
```



### ({a, b, ...rest} = obj);

* 这里是将前面没有进行赋值的对象全部加入了最后一个对象。
* 其次，最后一个是不会进入原型链进行查找赋值的。
* 如果没有了，那么便会变成空对象。

```
let obj = {
  a: 10,
  b: 20,
  c: 30
};
obj.__proto__ = {
  x: 1111,
}
let {a, ...res} = obj;
console.log(a, res);
```



### [a=4] / {a = 10} 解构默认值

* 简单来说就是设置一个默认值，不会变成undefined
* 对于对象也有效果 
* 默认值可以是对象。

```
var a, b;

[a=5, b=7] = [1];    //[a=5, b=7] = [1, undefined]; ,就算是这样也是 1， 7
console.log(a); // 1
console.log(b); // 7
```

```
({a=5, b=7} = {
  a: 3,
});
console.log(a); // 1
console.log(b); // 7

({a=5, b= { x: 1 }} = { a: 3 }); 并且还能使用赋值默认对象效果。
```



### [a, b] = [b, a]; 交换变量

* 交换的变量可以是对象，也可以是值类型。
* ({a, b} = {b, a});  这个没有交换变量的效果。

```
let a= {a: 4},
    b= {x:3};
[a, b] = [b, a];
console.log(a, b);
```



### 忽略值 [a, , b] = arr;

* 可以对不要的值不使用变量进行存储，使用 `, ,` 这样就能达到忽略的效果
* 对象不能使用忽略值的效果，会报错，
* `[, ,] = arr` ， 这样可以忽略全部返回值

```
function f() {
  return [1, 2, 3];
}

var [a, , b] = f();
console.log(a); // 1
console.log(b); // 3
```



### 命名赋值 {p: foo} = obj; 别名

* 只能用于对象
* 可以将其重新命名。
* 可以将命名赋值和默认值一同使用 {p:foo = 10}

```
var o = {p: 42, q: true};
var {p: foo, q: bar} = o;

console.log(foo); // 42
console.log(bar); // true 
```



### 方法参数的赋值 

* 首先，function fn(op = {}) {}

```
function drawES2015Chart({size = 'big', cords = { x: 0, y: 0 }, radius = 25} = {}) {}
上面这个写法很高级，但是有点难以理解。

function fn(o = {x: 'x', c: 1}) {}
这个写法，虽然可以理解了，但是如果传参时添加了一个参数便会全部覆盖，达不到想要的效果。

function fn(o = {}) {}
此时如果没有传入参数则会变成一个空对象，然后我们再将这个o换成一个对象

function fn({x = 3} = {}) {}
此时，如果不传入参数，那么先是一个空对象，然后再将空对象赋值给了前面的，然后就会生成一个 x=3 的数了。
如果传入的对象里面没有x，同样也是生成x=3的数。
所以此时再去看最开始那个就应该能够看懂了

function fn({x = 3}) {}
这里的区别，在于如果没有传入任何值，那么将会报错，而上面使用了空对象，可以不传递参数。
```

> 在上面的 **`drawES2015Chart`** 的函数签名中，解构的左手边被分配给右手边的空对象字面值：`{size = 'big', cords = {x: 0, y: 0}, radius = 25} = {}`。你也可以在没有右侧分配的情况下编写函数。但是，如果你忽略了右边的赋值，那么函数会在被调用的时候查找至少一个被提供的参数，而在当前的形式下，你可以直接调用 `**drawES2015Chart()**` 而不提供任何参数。如果你希望能够在不提供任何参数的情况下调用该函数，则当前的设计非常有用，而另一种方法在您确保将对象传递给函数时非常有用。



### 解构一个嵌套对象和数组

* 没啥讲的，但是却有点东西，对象别名使用了 []， 进行了数组赋值，然后再在里面进行了对象赋值，{}

```
const metadata = {
  title: 'Scratchpad',
  translations: [
    {
      locale: 'de',
      localization_tags: [],
      last_edit: '2014-04-14T08:43:37',
      url: '/de/docs/Tools/Scratchpad',
      title: 'JavaScript-Umgebung'
    }
  ],
  url: '/en-US/docs/Tools/Scratchpad'
};

let {
  title: englishTitle, // rename
  translations: [
    {
       title: localeTitle, // rename
    },
  ],
} = metadata;

console.log(englishTitle); // "Scratchpad"
console.log(localeTitle);  // "JavaScript-Umgebung"
```



### for of 迭代和解构

```
var people = [
  {
    name: 'Mike Smith',
    family: {
      mother: 'Jane Smith',
      father: 'Harry Smith',
      sister: 'Samantha Smith'
    },
    age: 35
  },
  {
    name: 'Tom Jones',
    family: {
      mother: 'Norah Jones',
      father: 'Richard Jones',
      brother: 'Howard Jones'
    },
    age: 25
  }
];

for (var {name: n, family: {father: f}} of people) {
  console.log('Name: ' + n + ', Father: ' + f);
}
```



### 计算属性名，可以被解构

* 简单来说，就是key变成了一个变量，然后使用[]， 来将一个常量换成了一个变量

```
let key = "z";
let { [key]: foo } = { z: "bar" };

console.log(foo); // "bar"
```



### 无效的标识符作为一个属性名

在这里，fizz-buzz 是一个无效的变量名，我们使用字符串形式，然后加上别名进行操作。

```
const foo = { 'fizz-buzz': true };
const { 'fizz-buzz': fizzBuzz } = foo;

console.log(fizzBuzz); // "true"
```

