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
https://cn.vuejs.org/v2/guide/
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



