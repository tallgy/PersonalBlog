---
title: Vue-methods使用箭头函数
date: 2021-10-31 16:51:55
tags:
 - Vue
 - 随笔
categories:
 - Vue
 - 随笔
---



#  Vue 

本篇随笔写的是在Vue的 **methods** 方法里面使用箭头函数。



在说明之前，我们先对这个进行一下分析：

首先 一个 **Vue** 的实例实则也是通过里面的对象进行的操作。

其次，**箭头函数的作用**，主要是 this 指向的不同，箭头函数的 this 指向是根据当前的上下文进行决定的。

所以首先可以认为，在Vue里面使用箭头函数，如果要使用 data里面 数据应该是不行，因为data里面的数据是通过 this 来获取，所以我们可以认为，Vue 内部对每个方法进行了一个this指向的变化，而箭头函数是无法修改的。

```
  var app4 = new Vue({
    el: '#app-4',
    data: {
      todos: [
        { text: '学习 JavaScript', flag: true },
        { text: '学习 Vue', flag: false },
        { text: '整个牛项目', flag: true }
      ]
    },
    methods: {
      test() {
        console.log(this.todos);
      },
      test1: () => {
        console.log(this.todos);
      }
    },
  })
```



**示例2：**

```
  var a = 3;

  let obj = {
    a: 1,
    m: {
      t: () => {
        console.log(this);
        console.log(this.a);
      },
      t1() {
        console.log(this);
        console.log(this.a);
      }
    }
  }
```

t 使用的箭头函数，this 的指向为 window

t1 使用的普通函数，this 的指向，指向了调用他的方法。



**结论：**

* 使用箭头函数的方法，确实不能获取到 data 的值
* 使用箭头函数后，this 的指向指向了 window 对象。
* 对于对象的对象的箭头函数，this的指向，还是指向的最外边的对象所处的上下文位置。