---
title: JavaScript—instanceof使用
date: 2023-05-26 14:57:36
tags:
categories:
---

# instancof

## 作用

```

用于检测某个构造函数是否存在于某个实列的原型链中
[source] instancof [target]
代表了 target 的 原型 是否存在于 source 的原型链中。

比如

const a = new Array();

console.log(a instanceof Array);  // true, 因为 Array.prototype 存在于 a 的原型链中
console.log(a instanceof Date); // false, 因为 a 的原型链中没有 Date.prototype 

```

## 简单实现

```

const myInstanceof = (left, right) => {
    // 取出右侧的原型
    const rightProto = right.prototype;
    // 先取出左侧实例的原型链上的第一个原型。
    let leftProto = left.__proto__;

    // 判断是否存在，没有说明了检测完了也没有存在，直接return false；
    while(leftProto) {
        // 相等，说明了 构造函数是存在于这个实例的原型链上的。
        if (leftProto === rightProto) {
            return true;
        }
        // 迭代赋值
        leftProto = leftProto.__proto__;
    }

    return false;
}

```