---
title: Vue-使用-基础使用
date: 2021-10-29 23:15:01
tags:
 - Vue
 - Vue2
 - 文档
categories:
 - Vue
 - Vue2文档
---



#  Vue的基本使用

```
简单入门教程
	https://cn.vuejs.org/v2/guide/
API
	https://cn.vuejs.org/v2/api/

在这里我就先进行一个简单的教程的学习。不过于深入了解。
```



## 引入

​		这里我们使用 script 进行引入

​		还可以使用 **npm** 进行下载引入 和 使用 **VueCLI** 脚手架，使用 npm 和 脚手架 的好处是，我们可以方便进行包管理。进行较大型应用时可以进行使用。但是我们这里的主要目的时了解 Vue 的基本使用和Vue 的一些规范，所以就直接使用 **script** 引入。

​		对于 **script** 引入，有两种：

* 第一种是直接进行引入。这种是创建了一个全局 Vue 的的实例，可以在后面进行使用，但是不能在前面进行使用。

  * ```
    <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
    ```

* 第二种是使用了 模块化 。 在引用以后，可以在改代码的后面直接进行使用。但是不能在 另外一个 script 标签内部使用，具体的原因是 使用了 module ，这个属于异步加载了。对于script 的异步我们后续在了解。

  * ```
    <script type="module">
      import Vue from 'https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.esm.browser.js'
      
      Vue.log;
    </script>
    ```

这里我们使用第一种引入。



## 声明式渲染

### Mustache 语法 {{ valueName }}

```
<div id="app">
  {{ message }}
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: 'test'
    }
  });
</script>
```

​		这里这个 message 是响应式的。如果我们在浏览器控制台修改了 message 的值，页面也会对应发生变化。里面的原因是使用了 **OBject.defineProperty()** 对于 Vue 的响应式。我们后续在了解 Vue源码的时候在进行讨论。

```
app.message = 'a';

页面会同时进行修改。
```

​		这里我们进行一个分析。

```
new Vue，这是一个 new 方法，会返回一个实例，里面的参数是一个对象，对象里面又是很多属性和对象的组成。

这里出现了。el 和 data。
```

```
其中el，是你要绑定的元素， 可以是 CSS 选择器，也可以是一个 HTMLElement 实例。
	参考：	https://cn.vuejs.org/v2/api/#el
CSS选择器： #app, .app, div.app ...
HTMLElement实例： 
	const span = document.querySelector('span.app');
	el: span.
其次，对于有多个满足的情况，只会对第一个进行编译。
```

```
data：
		我们可以看出，这个是一个data的对象。然后是对对象里面的数据进行了一个响应式的处理。然后我们也可以在后续在开发时可以发现，data是一个 返回的对象。这里是因为对象是使用的地址赋值，如果不通过return {}, 会让共同使用的组件会使用相同的数据。但是如果是return {}。那么在每次返回时都会返回一个新的对象出来。而不会共同使用一个对象。 
```



### 使用指令绑定 attribute (v-bind)

```
<span class="app" v-bind:title="message">
  {{ message }}
</span>
```

​		这里 v-bind 是 Vue 提供的 `attribute `。它可以绑定元素原有的 `attribute` 。这个指令的效果是： 将这个元素节点的 `title` attribute 和 Vue 实例的 `message` property 保持一致。

​		可以通过使用 app.message = 'new'; 可以发现，内容也随之更新。



## 条件与循环

### v-if

```
<div id="app-3">
  <p v-if="seen">现在你看到我了</p>
</div>


var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})

这里设置的 seen ， 设置为 true ，就会显示。设置为 false ，就不会显示。
```

​		可以通过设置 v-if 来进行这个标签的显示和隐藏。此外，Vue 也提供一个强大的过渡效果系统，可以在 Vue 插入/更新/移除元素时自动应用[过渡效果](https://cn.vuejs.org/v2/guide/transitions.html)。

​		这个过渡效果，我们后续在讲。

​		`v-if` 可以控制一个标签的显示和隐藏，还有 `v-show` 也有一样的效果。

**区别：**

* ```
  https://cn.vuejs.org/v2/guide/conditional.html#v-if-vs-v-show
  ```

* `v-if` 也是**惰性的**：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。
* 相比之下，`v-show` 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。
* 如果需要非常频繁地切换，则使用 `v-show` 较好；如果在运行时条件很少改变，则使用 `v-if` 较好。



### v-for

```
https://cn.vuejs.org/v2/api/#v-for
```

​		`v-for` 指令可以绑定数组的数据来渲染一个项目列表：

```
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>


var app4 = new Vue({
  el: '#app-4',
  data: {
    todos: [
      { text: '学习 JavaScript' },
      { text: '学习 Vue' },
      { text: '整个牛项目' }
    ]
  }
})

循环。 v-for="item in items"，  会循环items，赋值给 item。
```

**此外：**

​	在使用 **v-if** 搭配 v-for 时，**v-for** 的优先级会高于 **v-if**。

```
<li v-for="todo in todos" v-if="todo.flag">
```

​		在控制台里，输入 `app4.todos.push({ text: '新项目' })`，你会发现列表最后添加了一个新项目。

**注意：**

​	这里使用了 **push**， 方式添加了新项目，页面发生了改变，但是如果是 app4.todos[4] = xxx， 这样就不会发生页面的改变，这里是因为Vue 响应式的原因，至于原理，我们后续在讲解。想要提前知道的可以了解一下，Object.defineProperty()。



## 处理用户输入

​		用户和应用的交互，可以使用 **v-on** 指令来添加一个事件的监听器

这里的 v-on，就代表了 on

```
<button v-on:click="reverseMessage">反转消息</button>

<button onclick="reverseMessage()">反转消息</button>
```

```
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```

​		这里新添加了一个 **methods** 的对象，里面存放的是方法。如果是使用 **Vue** 的属性 **attribute**， **v-on** ，来进行的绑定方法，那么就需要将方法写在这个methods 里面。不写在 methods 里面的方法是没有效果的。

​		其次，这里面建议不要使用 箭头函数，因为箭头函数 的this指向是和 当前的上下文 相关的，所以在箭头函数里面使用不了data的方法。箭头函数的指向是全局。

​		在这里，我们只需要写上逻辑，不需要操作DOM，这个就是 MVVM 中， Vue 的 VM，我们只需要在 M和V 即可。



