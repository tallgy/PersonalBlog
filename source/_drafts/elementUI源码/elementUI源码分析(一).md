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

​		在 package/theme-chalk/src 里面。

​		common，里面创建一个 var.scss 里面存储了一些公用的变量。其他地方需要时要引入 var.scss

​		以及在一个index.scss 里面进行一个引入。

​		然后再看看 element ui的官网。

```
import 'element-ui/lib/theme-chalk/index.css';
这个里面就是存储了全局的css，然后进行一个引入。
```

​		并且将icon.scss 进行了引入，然后通过全局引入index来引入的icon。

​		原因：

* 因为 iconfont 的引入是需要使用url的引入

  * ```
    src: url('./fonts/iconfont.ttf?t=1643365814886') format('truetype');
    ```

* 同时对于引用的一个问题。那就是如果我在input目录下的index.scss 进行引入。那么对应的需要将引入的url路径进行改变，例子：url('../fonts/iconfont.ttf')

  * 因为执行文件在这个目录，所以最终scss编译为css的时候会将路径引入在这里。所以会出现问题，所以Elem UI 的方法是将其在外面创建一个index.scss，然后只需要引入这个index.scss，即可。

```
fonts
	iconfont.ttf
input
	index.scss
icon.scss
index.scss
```







## fonts-icon

```
package/theme-chalk/src/icon.scss
```

* 首先，这个fonts-icon 是使用的 font class 这个方法，具体的几个方法。我们可以通过去 阿里巴巴矢量图标库下载时，使用下载代码，查看demo的html来进行查看区分。

* 使用 @font-face 进行初始化。这个参数需要进行了解。这个大概就是说，指定一个font-family的名称，以及指定一个url代表了可以去哪进行获取。
  * 使用 #{$xx} 让 路径成为一个变量。
* 然后使用类名选择器进行查找类名，将对应类名的进行初始化项目。
* 目录构架，创建一个fonts的文件夹，用来存放 tff 和 字体的 js wtff 等文件，然后在外面创建一个icon的scss文件，用来进行使用。
* 然后对要调用的使用类进行调用，然后类将同一进行一个css选择器进行初始化和选择。



​		这里我们看一下代码

```
[class^="el-icon-"], [class*=" el-icon-"] {	}
```

​		这是element 写的选择器的选择，这两个选择器分别代表 选择从 el-icon- 开始 或者 包含了 ' el-icon-'，这里，你可能会有思考，为什么需要分开，如果我直接使用 class*="el-icon-" 这个不就是包含了吗。开始我也在思考，然后我发现了。因为 *= 是包含。那么如果用户自己定义的类名是 xxel-icon- 那么也会被我们这个包含进去，所以 这个就是需要进行分开，代表了是一个独立的class。



## input

```
package/inpu
```

​		Elem UI 使用 input的方法是。首先，通过type进行类型的决定，并且支持 textarea 类型。

​		输入框包含了 前/后 置元素 和 内容。 内容就是显示在输入框里面内部的icon等。 元素就是显示在输入框前面的。和输入框紧邻的一个样式。 

​		同时，禁用了组件的继承，然后将属性全部赋值于input 输入框里面，方便用于进行透明化的处理。这个常用于封装组件，并且组件的核心不处于根组件的情况。

```
v-bind="$attrs"

inheritAttrs: false
```

​		同时 Elem UI 的密码框是通过使用 show-password 进行的操作。

### icon 部分

​		并且使用了 前置内容 和 后置内容 通过使用 prefixIcon 和 suffixIcon 来进行设置。并且在里面同时也设置了 插槽。可以使用插槽的形式来配置前置和后置 内容。

```
<slot name="prefix"></slot>
<i class="el-input__icon"
  v-if="prefixIcon"
  :class="prefixIcon">
</i>
```

​		使用了 v-if 对于没有使用icon的进行了 隐藏。

​		如果使用了 prefixIcon 进行了图标，或者 $slots.prefix 插槽。

```
span
	v-if="prefixIcon || $slots.prefix"
```

### input事件部分

```
@compositionstart="handleCompositionStart"
@compositionupdate="handleCompositionUpdate"
@compositionend="handleCompositionEnd"
@input="handleInput"
@focus="handleFocus"
@blur="handleBlur"
@change="handleChange"
```









