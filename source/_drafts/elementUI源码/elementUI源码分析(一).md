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

**这里的问题点：** 

* 我们为什么不使用 $listeners 进行封装。

* 我们可以知道，默认情况下，进行组件方法的监听会加载入根组件

* 但是，因为组件内部的根组件不是input，所以对于监听一个原生的方法。我们使用.native。

* 并且直接监听给根组件并不会被当作一个原生的监听方法。对于根组件的监听，我们就可以通过 $listeners 进行监听。

* 那么我好奇，如果我 使用 v-model，然后，组件内部使用 v-bind='$attrs' v-on="$listeners" 那么是不是就应该也有对应的效果啊。

* 当然事实是不行，因为数据返回是一个 input event 事件。所以我看到了vue 官网是使用的 this.$emit('input', event.target.value) 然后就可以了。

* 虽然这里，可以使用 v-model 了，但是，有另一个问题，那就这样写，自身的input事件会不一样。

  * 首先 使用 @input="xxFn" 此时的第一个参数不是event事件了，而是子组件传递过来的参数
  * 其次，如果我们使用 focus.native 那么就会失败。但是因为input是一个可冒泡事件，所以input使用没有问题。

* 同时注意，此时的$event 是子组件传递的数据了。

  * ```
    inputListeners: function () {
      var vm = this
      // `Object.assign` 将所有的对象合并为一个新对象
      return Object.assign({},
        // 我们从父级添加所有的监听器
        this.$listeners,
        // 然后我们添加自定义监听器，
        // 或覆写一些监听器的行为
        {
          // 这里确保组件配合 `v-model` 的工作
          input: function (event) {
            vm.$emit('input', event.target.value)
          },
        }
      )
    }
    
    v-on="inputListeners"
    
    父组件的调用
    方法
    @input="inputFn('$event', $event)"
    inputFn(event, e) {
      console.log('event', event, e) // $event, value
    },
    ```

**Elem UI input 的监听事件之 compositionstart**： 

通过使用 isComposing 来判断是否处于中文输入情况。对于中文的输入不会提交，不会进行验证和发送。

其中我们发现在update的时候，调用了一个 isKorean 这里的方法就是里面有一个正则判断来判断你的输入是不是韩文的。

```
const text = event.target.value;
const lastCharacter = text[text.length - 1] || '';
this.isComposing = !isKorean(lastCharacter);

判断是不是韩文。
export function isKorean(text) {
  const reg = /([(\uAC00-\uD7AF)|(\u3130-\u318F)])+/gi;
  return reg.test(text);
}
```

​		同时我发现，Elem UI对于 compositionstart 事件没有进行 emit 提交。所以我这里给我的UI做了一个提交。不知道是不是应该有个什么问题。

​		同时，注意Elem UI input并没有对date进行封装样式，只是另写了一个组件给的date样式，内部input实则还是String。

​		输入框的 清除框通过v-if进行的判断显示隐藏。通过focus事件和mouseenter和mouseleave来解决 focus 和 hover 的值。

输入框的 placeholder 样式使用的 scss的 mixin 和 content 进行的混入。



## style sass

​		样式的编写。

### 对于 placeholder 

​		使用 混入 和 content 进行了 一次 封装，后续只需要混入 这个placeholder 方法就可以进行处理。当然，我发现对于input貌似不需要使用这个混入，直接使用 ::placeholder 这个方法就可以。

```
@mixin placeholder {
  &::-webkit-input-placeholder {
    @content
  }

  &::-moz-placeholder {
    @content
  }

  &:-ms-input-placeholder {
    @content
  }
}
```

### disabled

​		同时使用混入进行disabled的方法的处理。





