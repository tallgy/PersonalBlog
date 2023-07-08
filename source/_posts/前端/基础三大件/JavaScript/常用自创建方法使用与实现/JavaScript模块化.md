---
title: JavaScript模块化
date: 2022-03-08 22:21:48
tags:
 - JavaScript
 - 随笔
categories:
 - JavaScript
 - 随笔
---

#  JavaScript 模块化

​		我们可以知道，JavaScript原来是应用于浏览器端的，所以当时是没有模块化的概念的。

​		在模块化之前，JavaScript主要使用闭包和立即执行函数来将方法用命名空间来进行划分。

​		但是后来Nodejs的诞生，将JavaScript带入了服务端之后，模块化就变得不可或缺。此时CommonJS便由此而生了。



模块化：

* 后端：CommonJS 对应的 Node.js
* 前端：AMD 对应的 RequireJS  和 CMD 对应的 Sea.js
* ES6：ES6 的模块化规范。



## Nodejs 的 CommonJS

​	Nodejs 参照了 CommonJS 的规范，通过 require 和 exports 进行模块化。

​		关键词：require、exports、module.exports

```
简单的使用
const depend = require('../xxx')

module.exports.run = 1;

这里：我们省略了后缀，node将按照一下顺序进行查找。其中，如果x是包模块，那么会查找 package.json 的main属性
X.js
X.json
X.node
X/index.js
X/index.json
X/index.node
```

​	require 就是导入。

​	exports 就是导出的接口，是 module.exports 的一个引用，但是我们最终使用的时候还是使用的 module.exports。

​	所以，如果我们对 exports 的修改是一个对象的赋值，那么会导致exports会指向另外一个栈的地址，但是并不会修改到 model.exports。

​	所以如果我们要导出一个对象，那么我们就需要使用 module.exports，这样的修改才能造成影响。

​	CommonJS 的模块化，导出和缓存的步骤。

![nodejs-require](JavaScript模块化/nodejs-require.jpg)

​		我们会先判断引入的文件是否存在缓存区，同时判断是否是原生的模块，对于原生模块，回去判断是否在原生模块的缓存。然后最终就是加载和引入。



## AMD、CMD

AMD规范：RequireJS

​	AMD规范是异步的，通过define方法将代码定义为模块，在被require时加载依赖的模块。

​	AMD 是 "Asynchronous Module Definition" 的缩写，意思就是"异步模块定义"。

CMD规范：Sea.js



​		AMD和CMD的规范其实都差不多。只是AMD的一个异步的CMD是一个在书写上依照了CommonJS的一个规范。



## ES6

​		ES6模块化，Es6在后续也出现了模块化，就是

import，exports



* 和commonjs的区别：

​		CommonJS 只能在运行时确定导出的接口，实际导出的就是一个对象。而 ES6 Module 的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及导入和导出的变量，也就是所谓的"编译时加载"。

​		正因为如此，`import` 命令具有提升效果，会提升到整个模块的头部，首先执行。

​		简单来说就是，我虽然没有先import，但是我还是会提前import。就像是function写在下面也可以调用一样。

​		也正因为 ES6 Module 是编译时加载， 所以不能使用表达式和变量，因为这些是只有在运行时才能得到结果的语法结构。

​		就是，不存在变量的使用情况，因为此时还没有。

​		commonjs 是深拷贝，而es6是浅拷贝



