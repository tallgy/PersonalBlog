---
title: JavaScript变量的作用域
date: 2021-11-10 20:18:30
tags:
 - JavaScript
 - 随笔
categories:
 - JavaScript
 - 随笔
---



#  JavaScript变量的作用域

* 浏览器和node的一个小区别，在全局变量中，

  * node的全局变量，是一个名为Global 的全局对象，并且使用 var定义的变量不会加入其中，只有直接定义的变量会加入其中。

    * ```
      var a = 1;
      b = 2;
      ```

  * 浏览器的全局变量，是一个window对象，使用var定义的变量和直接定义的变量也会加入其中。

    * ```
      a = 1;
      var b = 2;
      
      下面定义的不会加入其中。
      let c = 3;
      const d = 4;
      ```

