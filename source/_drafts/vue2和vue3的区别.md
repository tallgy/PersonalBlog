---
title: Vue2和Vue3的区别
date: 2021-10-16 20:02:19
tag: 
 - vue
 - 面试
---



# Vue2和Vue3的区别



## 从内部上



### 响应式的改变

Vue2是使用的 `Object.definePropert()` ，而Vue3是使用的 `Proxy` 来对数据进行的代理。

**definePropert的使用**

```
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
```

```
https://www.jianshu.com/p/8fe1382ba135
```

```
let a = {};

//这里这个bV是用来存储属性的值。
let bV;
Object.defineProperty(a, 'b', {
  set(v) {
    console.log('set', v);
    bV = v;
  },
  get() {
    console.log('get', bV);
    return bV;
  },
  enumerable : true,
  configurable : true
});
```

**prosy**

```
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
```

```
let validator = {
  set(obj, prop, value) {
    console.log('set   ');
    console.log(obj, prop, value);
    obj[prop] = value;
    return true;
  },
  get(obj, prop) {
    console.log('get   ');
    console.log(obj, prop);
    return obj[prop];
  }
}

let person = new Proxy({}, validator);

person.a = 100;
console.log(person.a);
```

proxy的优势

```
使用对象，可以是数组，函数，任何对象，而definePropert只能是一个对象的某一个值进行监听。

defineProperty只能监听某个属性，不能对全对象监听
可以省去for in、闭包等内容来提升效率（直接绑定整个对象即可）
可以监听数组，不用再去单独的对数组做特异性操作 vue3.x可以检测到数组内部数据的变化
```



### 默认进行懒观察（lazy observation）。

```
在 2.x 版本里，不管数据多大，都会在一开始就为其创建观察者。当数据很大时，这可能会在页面载入时造成明显的性能压力。3.x 版本，只会对「被用于渲染初始可见部分的数据」创建观察者，而且 3.x 的观察者更高效。
```



### 更精准的变更通知。

```
2.x 版本中，使用 Vue.set 来给对象新增一个属性时，这个对象的所有 watcher 都会重新运行；3.x 版本中，只有依赖那个属性的 watcher 才会重新运行。
```



## 从使用上

```
https://v3.cn.vuejs.org/guide/migration/introduction.html
```



### 组合式API

**setup**

注意

```
在 setup 中你应该避免使用 this，因为它不会找到组件实例。
setup 的调用发生在 data property、computed property 或 methods 被解析之前，所以它们无法在 setup 中被获取。
```

使用ref将值封装于对象之中。

```
因为在 JavaScript 中，Number 或 String 等基本类型是通过值而非引用传递的，所以封装成对象之后变有了作用。
```

在setup里面注册生命周期钩子

```
onMounted(getUserRepositories) // 在 `mounted` 时调用 `getUserRepositories`

这个代码放在setup内部
```



### Teleport

Teleport 提供了一种干净的方法，允许我们控制在 DOM 中哪个父节点下渲染了 HTML，而不必求助于全局状态或将其拆分为两个组件。

```
<teleport to="body"></teleport>
```

特点：使用之后， `to="body"`，代表了在渲染时会作为body的子元素来进行渲染，不会出现一些，因为父元素的设置定位等问题而影响到了 `teleport ` 的渲染。



### 片段

Vue2.x 不支持多根节点，而Vue3.x支持了。

在 3.x 中，组件可以包含多个根节点！但是，这要求开发者显式定义 attribute 应该分布在哪里。



**为什么Vue实例只能有一个根元素**

```
https://zhuanlan.zhihu.com/p/111691226
```

从源码上来说。

```
在实例化Vue的时候，填写一个el选项，来指定入口
在Vue的源码里面，他只寻找了指定选择器的第一个元素。所以后面的都会原封不动。
```

对于单文件组件。

template里面如果有 多个div入口，不知道如何指定 谁作为入口。



### 单文件组件`<script setup>`



<script setup> 是在单文件组件 (SFC) 中使用组合式 API 的编译时语法糖。相比于普通的 <script> 语法，它具有更多优势：
	更少的样板内容，更简洁的代码。
    能够使用纯 Typescript 声明 props 和抛出事件。
    更好的运行时性能 (其模板会被编译成与其同一作用域的渲染函数，没有任何的中间代理)。
    更好的 IDE 类型推断性能 (减少语言服务器从代码中抽离类型的工作)。



```
里面的代码会被编译成组件 setup() 函数的内容。这意味着与普通的 <script> 只在组件被首次引入的时候执行一次不同，<script setup> 中的代码会在每次组件实例被创建的时候执行。
```



**限制：没有 Src 导入**

```
由于模块执行语义的差异，`<script setup>` 中的代码依赖单文件组件的上下文。当将其移动到外部的 `.js` 或者 `.ts` 文件中的时候，对于开发者和工具来说都会感到混乱。因而 **`<script setup>`** 不能和 `src` attribute 一起使用。
```



### 状态驱动的动态 CSS

单文件组件的 `<style>` 标签可以通过 `v-bind` 这一 CSS 函数将 CSS 的值关联到动态的组件状态上：



```
script 
	export default
		data() {
			return {
				color: 'red'
			}
		}


<style>
.text {
  color: v-bind(color);
}
</style>
```



### Suspense**新增**

Suspense 是一个试验性的新特性，其 API 可能随时会发生变动。特此声明，以便社区能够为当前的实现提供反馈。

生产环境请勿使用。

```
	在正确渲染组件之前进行一些异步请求是很常见的事。组件通常会在本地处理这种逻辑，绝大多数情况下这是非常完美的做法。
	该 `<suspense>` 组件提供了另一个方案，允许将等待过程提升到组件树中处理，而不是在单个组件中。
```

