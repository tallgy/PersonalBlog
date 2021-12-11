---
title: Vue-使用-可复用性和组合
date: 2021-10-29 23:15:35
tags:
 - Vue
 - Vue2
 - 文档
categories:
 - Vue
 - Vue2文档
---



#  Vue2教程-可复用性和组合

## 混入

### 基础

​		混入（mixin）提供了一种非常灵活的方式。用来分发Vue组件中的可复用的功能。当组件使用混入对象时，所有混入对象的选项将 ‘混合’ 进入该组件本身的选项。

​		简单来说就是有一个对象叫做，mixins

```
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// 定义一个使用混入对象的组件
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // => "hello from mixin!"
```



### 选项合并

​		当组件和混入对象含有同名的选项时，这些选项将以恰当的方式进行 合并。数据对象会在内部进行递归的合并，对于发生的冲突以组件数据优先。

```
const mixin = {
	data() {
		return {
			me: {
				a: 1,
				b: 12,
			}
		}
	}
}

new Vue() {
	mixins: [mixin],
	data() {
		return {
			me: {
				a: 123,
				d: 12341,
			}
		}
	}
}
使用了 mixin 混入，将数据递归的进行覆盖。 a 123，b 12，d 12341
```

​		对于钩子函数，将会合并为数组进行全部调用。

​		值为对象的选项，一般都是进行递归合并操作。当出现键名冲突，一般以组件为主。

​		注意：`Vue.extend()` 也使用同样的策略进行合并。



### 全局混入

​		混入也是可以使用全局进行注册的。但是要格外小心。一旦使用了全局混入。就将会影响每一个之后创建的Vue实例。

```
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

此时对于如果使用组件时存在 option的对象，那么将会输出。
```



> 请谨慎使用全局混入，因为它会影响每个单独创建的 Vue 实例 (包括第三方组件)。大多数情况下，只应当应用于自定义选项，就像上面示例一样。推荐将其作为[插件](https://cn.vuejs.org/v2/guide/plugins.html)发布，以避免重复应用混入。



### 自定义选项合并策略

​		



## 自定义指令

## 渲染函数 和 JSX

## 插件

## 过滤器
