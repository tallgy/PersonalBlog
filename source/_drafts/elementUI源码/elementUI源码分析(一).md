---
title: elementUI源码分析(一)
date: 2022-01-29 17:18:30
tags:
 - elementUI
categories:
 - elementUI
---



# elementUI 源码分析

​	最近想学一下如何写一个组件，所以学习一下elementUI的源码，大概有一个基本的了解。



## 入口 package.json

​	主要的几个地方

```
main，定义一个入口文件，默认值是模块根目录的index.js
style，打包工具通过style知道了样式文件的打包位置
```



## src/index.js

​	我们看一下 src的index

​	其实大致的意思就是，先进行组件的引入，然后将引入的组件合并为一个数组，然后创建一个 install 方法。里面的核心就是 Vue.component 方法。最后在export 导出组件和install方法。

​	install方法为什么需要呢，因为在调用 Vue.use 时，如果插件是一个对象，那么就会查找里面的install方法。如果插件是一个函数，那么就会作为install方法进行调用。

```
import xx from 'xxx'

const c = [
	xx
]

const install = function(Vue, opts = {}) {
	c.forEach(v => {
		Vue.component(v.name, v)
	})
}

export default {
	install,
	
	xx
}
```

这里，我个人觉得使用 对象来代替数组进行存储。可以将后续的export default 使用解构赋值来进行代替。

```
const c = {
	xx
}

const install = function(Vue, opts = {}) {
	for (const key in c) {
		Vue.component(key, c[key])
	}
}

export default {
	install,
	
	...c
}
```



## 组件的文件格式

创建一个目录，然后创建一个index.js文件作为一个入口，创建一个src文件夹作为组件的存放。

```
xx
	src
		xx.vue
	index.js
```



### index.js 

其中，index.js 才是比较重要的一个部分

因为在使用 from 进行引入时，如果是一个目录，那么默认就会去查找index.js文件，当然如果配置了package.json的入口会不一样。

​	index.js的内容，简单来说就是创建了一个install方法。

```
import xx from '../src/xx'

xx.install = function(Vue) {
	Vue.component(xx.name, xx)
}

export defalt xx
```



## index.scss

在 package/theme-chalk/src 里面。

common，里面创建一个 var.scss 里面存储了一些公用的变量

其他地方需要时要引入 var.scss

以及在一个index.scss 里面进行一个引入。

然后再看看 element ui的官网。

```
import 'element-ui/lib/theme-chalk/index.css';
这个里面就是存储了全局的css，然后进行一个引入。
```



## fonts-icon

​	package/theme-chalk/src/icon.scss

使用 @font-face 进行初始化。这个参数需要进行了解。

使用 #{$xx} 让 路径成为一个变量。

然后使用类名选择器进行查找类名，将对应类名的进行初始化项目。



