---
title: vue-自定义指令-loading的使用
date: 2022-04-04 01:42:56
tags:
 - Vue
categories:
 - Vue
---

# vue-自定义指令-loading的使用

​		首先，本片文章的核心其实就是 element-ui 的 v-loading 的一个源码查看 以及实现，因为组件需要使用 loading 加载。

​		这里就简单的说一些核心的部分内容



## 自定义指令

​		首先就是，自定义指令。vue支持自定义指令，同时我们可以看到 element-ui 实际上使用的就是自定义指令。Vue.directive

```
https://cn.vuejs.org/v2/guide/custom-directive.html
```

​		这个是一个官网的链接，简单说明了自定义指令的部分。知道自定义指令的一个大概流程。

​	简单来说，我们就是要简易实现一个 v-loading。然后我们查看钩子函数。大概就是需要 bind 和 unbind 两个。钩子函数。

```
// 注册一个全局自定义指令 `v-loading`
Vue.directive('loading', {
  // 当被绑定的元素插入到 DOM 中时……
  bind() {}
  unbind() {}
})
```



​		其实到了这里就已经差不多都知道了，大概就是在 bind 的时候通过js进行添加dom，然后再 unbind 的时候进行移除。

​		我这里实现了比较核心的几个部分。

* 先使用 extend 创建一个类，然后我们通过创建这个类的实例创建dom。

```
const Mask = Vue.extend(Loading)

Vue.directive('loading', {
    bind(el, binding, vnode) {

      const mask = new Mask({
      	// 代表挂载在一个新的 div 节点上面
        el: document.createElement('div')
      })
      // $el 就是 实例的根元素
      el.mask = mask.$el;
			// 然后我们将这个 添加在 el 上面。
			// 这里的 el 就是在 使用了 v-loading 的根节点。
      el.appendChild(el.mask)
    },
    unbind(el, binding, vnode) {

    }
  })
```

```
// 然后我们可以通过 这个use 方法进行创建。创建插件。
Vue.use(directive);
```

