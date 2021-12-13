---
title: Vue-使用-规模化
date: 2021-10-29 23:15:47
tags:
 - Vue
 - Vue2
 - 文档
categories:
 - Vue
 - Vue2
---



#  Vue2教程-规模化

## 路由

### 官方路由

​		vue-router

​	对于大多数单页面应用，都推荐使用官方支持的 [vue-router 库](https://github.com/vuejs/vue-router)。更多细节可以移步 [vue-router 文档](https://router.vuejs.org/)。



### 从零开始的简单路由

​		将vue实例存储，然后通过一个 routes 对象，对象存储了vue实例。然后通过 window.location.pathname 来进行获取url，通过判断 url 来决定显示什么页面，

```
const NotFound = { template: '<p>Page not found</p>' }
const Home = { template: '<p>home page</p>' }
const About = { template: '<p>about page</p>' }

const routes = {
  '/': Home,
  '/about': About
}

new Vue({
  el: '#app',
  data: {
    currentRoute: window.location.pathname
  },
  computed: {
    ViewComponent () {
      return routes[this.currentRoute] || NotFound
    }
  },
  render (h) { return h(this.ViewComponent) }
})
```

​	唯一的好奇点就是。render函数也是一个会多次执行的，比如这里，return h(this.ViewComponent) 这里 viewcomponent 变化了，然后这个render也重新执行了一次。性质应该和computed 计算属性一样。是一个可以进行缓存的一个方法。



## 状态管理

### 类Flux状态管理的官方实现

#### React的开发者参考一下信息

### 简单状态管理起步使用

​		因为vue的data是单独的，不共享的，但是对于有些数据，我们需要进行共享，在a发生改变了之后，b也会发生改变，比如开始的 props 和 $emit()，但是，对于这个问题，我们发现这个的耦合过高，并且是针对于父子组件，如果组件的层级过高，需要再使用 **依赖注入** 进行跨组件使用。

​	下面这个例子就是进行的调用。他们共享了 sourceOfTruth，当A变更了 sourceOfTruth ，B也会自动的更新他们的视图。

​	但是，这样使用在调试的时候变为噩梦。因为你不知道变更出现在哪里。也不知道发生了什么变更。

```
const sourceOfTruth = {}

const vmA = new Vue({
	data: {
		sourceOfTruth
	}
})
const vmB = new Vue({
	data: {
		sourceOfTruth
	}
})
```

​		于是，对应的，我们便采用了一个简单的 store 模式。

```
const store = {
	debug: true,
	state: {
		message: 'Hello!'
	},
	setMessageAction(newValue) {
		if (this.debug) log('set message action triggered with', newValue)
		this.state.message = newValue
	},
	clearMessageAction() {
		if (this.debug) log('clear message action triggrered')
		this.state.message = ''
	}
}
```

```
const vmA = new Vue({
	data: {
		privateState: {},
		sharedState: store.state
	}
})
```

> 重要的是，注意你不应该在 action 中 替换原始的状态对象 - 组件和 store 需要引用同一个共享对象，变更才能够被观察到。

​		于是我们便延申了约定，组件不能直接变更 state，而是要执行action 来 dispatch 事件通知 store改变state。最终便达成了 flux 架构。



## 服务端渲染

### SSR完全指南

​		在 2.3 发布后我们发布了一份完整的构建 Vue 服务端渲染应用的指南。这份指南非常深入，适合已经熟悉 Vue、webpack 和 Node.js 开发的开发者阅读。请移步 [ssr.vuejs.org](https://ssr.vuejs.org/zh/)。

### Nuxt.js

​		从头搭建一个服务端渲染的应用是相当复杂的。幸运的是，我们有一个优秀的社区项目 [Nuxt.js](https://nuxtjs.org/) 让这一切变得非常简单。Nuxt 是一个基于 Vue 生态的更高层的框架，为开发服务端渲染的 Vue 应用提供了极其便利的开发体验。更酷的是，你甚至可以用它来做为静态站生成器。推荐尝试。

### Quasar Framework SSR + PWA

​		[Quasar Framework](https://quasar.dev/) 可以通过其一流的构建系统、合理的配置和开发者扩展性生成 (可选地和 PWA 互通的) SSR 应用，让你的想法的设计和构建变得轻而易举。你可以在服务端挑选执行超过上百款遵循“Material Design 2.0”的组件，并在浏览器端可用。你甚至可以管理网站的 `<meta>` 标签。Quasar 是一个基于 Node.js 和 webpack 的开发环境，它可以通过一套代码完成 SPA、PWA、SSR、Electron、Capacitor 和 Cordova 应用的快速开发。



## 安全

