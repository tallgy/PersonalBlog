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



# 介绍

```
https://cn.vuejs.org/v2/guide/index.html
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

### Mustache 语法 { { valueName }}

```
<div id="app">
  { { message }}
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
  { { message }}
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
      { { todo.text }}
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

​		在这里，我们只需要写上逻辑，不需要操作DOM，这个就是 MVVM 中， Vue 的 VM，我们只需要在 M和V 上进行操作即可。



## v-model 实现双向绑定

​		v-bind，可以实现单向的绑定，就是指可以通过修改data数据来进行页面的修改。

​		但是，如果是对于一个 input 的输入框呢？

​		我们可以将 data 数据绑定到输入框作为一个初始值。

​		但是我们可以在对输入框进行输入时，同时修改 data 的数据吗。显然是不行的。所以就有了 v-model

​		v-model ，它可以进行数据的双向绑定，不但用户的输入会修改 data， data 的变化也会影响视图。

```
<input v-model="message">
```

​		通过这个我们 可以看出，v-model 没有说使用value，还是什么，但是却有效果，这里是因为 v-model 会根据控件的类型自动选取正确的方法来更新。限制：	input， select， textarea， components

```
https://cn.vuejs.org/v2/api/#v-model
```



## 组件化应用构建

​		Vue 的另一个重要的概念就是 **组件化** 。几乎任意应用界面都可以抽象为一个组件树。

​		在 Vue 中注册一个组件

```
// 定义名为 todo-item 的新组件
Vue.component('todo-item', {
  template: '<li>这是个待办项</li>'
})

var app = new Vue(...)
```

**示例：**

```
<div id="app-6">
  <my-item></my-item>
</div>


  Vue.component('my-item', {
    template: '<li>这是个待办项</li>'
  });

  var app6 = new Vue({ el: '#app-6' })
```

**注意点：**

* 使用的方式是，需要在 new 的实例的 Vue 的内部进行调用，因为在 实例内部，你写的方式才会被 Vue 所编译，不然是不会被 Vue 编译的。

* 其次，注册的组件需要在你 new 的实例前面，因为你在编译组件的时候要使用自定义的组件。所以，如果不在之前进行解析的话，就解析不了了。



## 组件传值

```
<div id="app-6">
  <my-item :todo="message"></my-item>
</div>


Vue.component('my-item', {
  props: ['todo'],
  template: '<li>这是个待办项 { { todo }}</li>',
});

var app6 = new Vue({
  el: '#app-6',
  data: {
  	message: false
  }
})
```

​		这里， props 是代表了获取到组件属性传递过来的值，在组件使用时，添加属性，这个属性是和 props 里面的名字相等。这里就是 todo， 其次， **:todo**，是一个语法糖，代表了 **v-bind:todo**，所以就会把 message 的值传递给 todo，然后再传递给 my-item。



# Vue实例

```
https://cn.vuejs.org/v2/guide/instance.html
```



## 创建一个Vue实例

​		每个 Vue 应用都是通过用 `Vue` 函数创建一个新的 **Vue 实例**开始的：

```
var vm = new Vue({
  // 选项
})
```

​		虽然没有完全遵循 [MVVM 模型](https://zh.wikipedia.org/wiki/MVVM)，但是 Vue 的设计也受到了它的启发。因此在文档中经常会使用 `vm` (ViewModel 的缩写) 这个变量名表示 Vue 实例。

​		至于为什么 Vue 没有完全遵循 MVVM 的原因是：Vue 中有一个属性，ref，这个属性可以拿到 DOM 对象，直接操作视图，所以违背了 MVVM。

​		在创建一个 Vue 实例时，你可以传入一个 [选项对象]([https://cn.vuejs.org/v2/api/#%E9%80%89%E9%A1%B9-%E6%95%B0%E6%8D%AE](https://cn.vuejs.org/v2/api/#选项-数据)) (data，methods...)。通过这些选项对象来创建你想要的行为。

​		一个 Vue 应用由一个通过 `new Vue` 创建的**根 Vue 实例**，以及可选的嵌套的、可复用的组件树组成。



## 数据与方法

​		当一个 Vue 实例被创建时，它将 `data` 对象中的所有的 property 加入到 Vue 的**响应式系统**中。当这些 property 的值发生改变时，视图将会产生“响应”，即匹配更新为新的值。

​		大致就可以理解为，需要将数据放在了 data 里面，Vue 就会自动创建响应式。

```
// 我们的数据对象
var data = { a: 1 }

// 该对象被加入到一个 Vue 实例中
var vm = new Vue({
  data: data
})

// 获得这个实例上的 property
// 返回源数据中对应的字段
vm.a == data.a // => true

// 设置 property 也会影响到原始数据
vm.a = 2
data.a // => 2

// ……反之亦然
data.a = 3
vm.a // => 3
```

​		通过上面我们可以知道，data的数据是直接可以通过 实例的返回来直接获取，数据是直接暴露于实例的顶层。理所当然，methods 的方法也是一样，所以我们会思考，如果方法名和数据名重合会怎么办。

```
  var app6 = new Vue({
    el: '#app-6',
    data: {
      message: false
    },
    methods: {
      message() {
        console.log(1);
      }
    },
  })
  
通过上面的案例可以看出，方法名是和 data 名称重合，所以在 data 已经创建了数据之后，方法创建会抛出异常。方法创建失败。
```

```
  而如果是一个普通的对象，后面的会将前面的覆盖掉。
  
  let obj = {
    a: 1,
    a() {
      console.log(2);
    }
  }
  console.log(obj);
```

​		其次

```
使用 vm.b = 1;
这种后续添加数据的方式是不会变成响应式的。
以及，使用了 Object.freeze() 也会阻止修改现有的 property，也意味着响应系统无法再追踪变化。
```

​		除了数据 property，Vue 实例还暴露了一些有用的实例 property 与方法。它们都有前缀 `$`，以便与用户定义的 property 区分开来。

```
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // => true
vm.$el === document.getElementById('example') // => true

// $watch 是一个实例方法
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
```

​		你可以在 [API 参考](https://cn.vuejs.org/v2/api/#实例-property)中查阅到完整的实例 property 和方法的列表。



## 实例生命周期钩子

​		生命周期钩子简单来说就是一个回调函数。在 Vue 在执行到每个过程的时候也会执行这些生命周期的钩子。

​		**举个栗子：**

```
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` 指向 vm 实例
    console.log('a is: ' + this.a)
  }
})
// => "a is: 1"

在 created 钩子可以用来在一个实例被创建之后执行代码：
```

**注意：**

> 不要在选项 property 或回调上使用[箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，比如 `created: () => console.log(this.a)` 或 `vm.$watch('a', newValue => this.myMethod())`。因为箭头函数并没有 `this`，`this` 会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 `Uncaught TypeError: Cannot read property of undefined` 或 `Uncaught TypeError: this.myMethod is not a function` 之类的错误。



## 生命周期图示

![lifecycle](Vue2-教程-基础使用/lifecycle.png)



# 模板语法

```
https://cn.vuejs.org/v2/guide/syntax.html
```

## 插值

**文本：**

​		数据绑定最常见的形式就是使用“Mustache”语法 (双大括号) 的文本插值：

```
<span>Message: { { msg }}</span>
```

这里 msg 会替代为 数据对象的 msg。并且还带有响应式的功能。

​		通过使用 [v-once 指令](https://cn.vuejs.org/v2/api/#v-once)，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上的其它数据绑定：

```
<span v-once>这个将不会改变: { { msg }}</span>
```



**原始HTML：**

​		双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 [`v-html` 指令](https://cn.vuejs.org/v2/api/#v-html)：

```
<span v-html="rawHtml"></span>
```

​		会将 span 的内容替换为 rawHtml。并且在里面不会解析 proterty。

> 你的站点上动态渲染的任意 HTML 可能会非常危险，因为它很容易导致 [XSS 攻击](https://en.wikipedia.org/wiki/Cross-site_scripting)。请只对可信内容使用 HTML 插值，**绝不要**对用户提供的内容使用插值。



**Attribute：（属性）**

​		Mustache 语法不能作用在 HTML 标签的属性上，所以要使用 v-bind 指令。

```
<div v-bind:id="dynamicId"></div>

对于同时使用了 v-bind:id 和 id 的。我们可以发现，谁在后面，其结果就是谁。
```

​		对于布尔值的 attribute，原生的HTML中，只要存在就意味着值为 true，而 v-bind，工作起来当值为 false，null等，甚至不会渲染。



**使用 JavaScript 表达式：**

​		对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持。

```
{ { number + 1 }}

{ { ok ? 'YES' : 'NO' }}

{ { message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```



## 指令

​		指令 (Directives) 是带有 `v-` 前缀的特殊 attribute。

**参数：**

​		一些指令能够接收一个“参数”，在指令名称之后以冒号表示。例如，`v-bind` 指令可以用于响应式地更新 HTML attribute：

```
<a v-bind:href="url">...</a>

此时这个url是和数据的url是绑定的。
```

​		`v-on` 指令，它用于监听 DOM 事件：

```
<a v-on:click="doSomething">...</a>
同时也有 mouseover 等等。
```

**注意：**

​		使用 v-on 指令监听 DOM 事件，原生的 onclick 方法会先于 v-on 进行监听，其次这个 v-on 里面的方法，既可以是 methods 的，也可以是 data 的。但是建议写在 methods 中。

```
<div id="app-6">
  <button @click="test1" onclick="console.log(1);">Button</button>
</div>
//其中这里这个 @ 代表了 v-on 的语法糖，我们后续会讲。

	let obj = {
    message: true,
    test1() {
      console.log(3);
    }
  }

  var app6 = new Vue({
    el: '#app-6',
    data: obj,
    methods: {
      test() {
        console.log(2);
      }
    }
  })
```



**动态参数： 2.6.0新增**

​		可以用方括号括起来的 JavaScript 表达式作为一个指令的参数：

```
<!--
注意，参数表达式的写法存在一些约束，如之后的“对动态参数表达式的约束”章节所述。
-->
<a v-bind:[attributeName]="url"> ... </a>

<a v-on:[eventName]="doSomething"> ... </a>
@[]，也可以。
```

​		这里的 `attributeName` 会被作为一个 JavaScript 表达式进行动态求值，求得的值将会作为最终的参数来使用。这里的Vue实例中有 data property attributeName，值为 href，则就等价于 v-bind:href="url"

​	**对动态参数的值的约束**

​		动态参数预期会求出一个字符串，异常情况下值为 `null`。这个特殊的 `null` 值可以被显性地用于移除绑定。任何其它非字符串类型的值都将会触发一个警告。

​	**对动态参数表达式的约束**

​		动态参数表达式有一些语法约束，因为某些字符，如空格和引号，放在 HTML attribute 名里是无效的。例如：

```
<!-- 这会触发一个编译警告 -->
<a v-bind:['foo' + bar]="value"> ... </a>

使用引号会无法编译。
并且使用了空格也会无法编译。
```

​		变通的办法是使用没有空格或引号的表达式，或用计算属性替代这种复杂表达式。

​		在 DOM 中使用模板时 (直接在一个 HTML 文件里撰写模板)，还需要避免使用大写字符来命名键名，因为浏览器会把 attribute 名全部强制转为小写：

```
<!--
在 DOM 中使用模板时这段代码会被转换为 `v-bind:[someattr]`。
除非在实例中有一个名为“someattr”的 property，否则代码不会工作。
-->
<a v-bind:[someAttr]="value"> ... </a>
```



**修饰符：**

​		修饰符 (modifier) 是以半角句号 `.` 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。例如，`.prevent` 修饰符告诉 `v-on` 指令对于触发的事件调用 `event.preventDefault()`：

​		与之相应的还有 `.laze` `.once` 等等。我们后续进行讲解。



## 缩写 语法糖

**v-bind 缩写**：

```
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>

<!-- 动态参数的缩写 (2.6.0+) -->
<a :[key]="url"> ... </a>
```



**v-on 缩写：**

```
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>

<!-- 动态参数的缩写 (2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```



# 计算属性和侦听器

```
https://cn.vuejs.org/v2/guide/computed.html
```



## 计算属性

​		简单来说，就是将逻辑更深层的解耦，比如：

```
{ { message.split('').reverse().join('') }}
```

​		在模板中放入太多的逻辑会让模板过重且难以维护。

​		在这个地方，模板不再是简单的声明式逻辑。你必须看一段时间才能意识到，这里是想要显示变量 `message` 的翻转字符串。当你想要在模板中的多处包含此翻转字符串时，就会更加难以处理。

​		所以，对于任何复杂逻辑，你都应当使用**计算属性**。

​		我认为，从一个开发来看，对于一个表达式，如果以后会有多个地方进行相同的逻辑的使用，就应当使用计算属性，方便维护。



### 基础例子

```
<div id="app-6">
  { { message }}
  <br>
  { { reversedMessage }}
</div>
```

```
  const vm = new Vue({
    el: '#app-6',
    data: {
      message: 'true',
    },
    computed: {
      // 计算属性的 getter
      reversedMessage: function () {
        // `this` 指向 vm 实例
        return this.message.split('').reverse().join('')
      }
    },
  })
```

​		这里我们声明了一个计算属性 `reversedMessage`。我们提供的函数将用作 property `vm.reversedMessage` 的 getter 函数

​		计算属性默认是的方法是一个getter 方法， 就像是使用了 `Object.defineProperty` 的getter一样进行了操作。



## 计算属性缓存 VS 方法

​		我们也可以发现，可以在插值表达式中使用方法来获取同样的效果。

```
<p>{ { reversedMessage() }}</p>

methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

​		首先，对于结果来说是完全相同的。不同的地方在于，**计算属性是基于它们的响应式依赖进行缓存的**。意思就是说，只有相关的响应式依赖发生了改变，他们才会重新求值。没有发生改变，多次使用计算属性会立即返回之前的结果。

​		**举个栗子：**

```
<div id="app-6">
  { { message }}
  <br>
  { { reversedMessage }}
  <br>
  { { reversedMessage }}
  <br>
  { { reversedMessage }}
</div>


  const vm = new Vue({
    el: '#app-6',
    data: {
      message: 'true',
    },
    computed: {
      // 计算属性的 getter
      reversedMessage: function () {
        // `this` 指向 vm 实例
        console.log(1);
        return this.message.split('').reverse().join('')
      }
    },
  })
```

​		我这里使用了很多个插值表达式，但是发现控制台的输出，只有一个，这里代表了后续是直接使用的之前的计算结果。

​		其次，在值发生变化之时，也只输出了一次。因此计算属性的缓存效果则比方法有了更好的性能。

​		当然，如果不希望有缓存的存在，可以使用方法来替代。



## 计算属性 VS 侦听属性

​		Vue 提供了一种更通用的方式来观察和响应 Vue 实例上的数据变动：**侦听属性**。 `watch`  。

​		侦听属性 和 计算属性的不同

```
watch: {
  firstName: function (val) {
  	this.fullName = val + ' ' + this.lastName
  },
  lastName: function (val) {
  	this.fullName = this.firstName + ' ' + val
  }
},
computed: {
  fullName: function () {
  	return this.firstName + ' ' + this.lastName
  }
}
```

​		从上面可以看出，侦听属性(watch)的特点是，当一个属性发生改变后，调用的方法。

​		其次，需要对其进行初始化，因为在最开始侦听属性不会进行调用。

​		最后，这个侦听属性的执行时机，我们通过一个简单的死循环就可以看出。侦听属性在 DOM 的变化之前。但是处于值的变化之后。起码下面这个情况满足。这个说法。

```
watch: {
  message: function (val) {
  console.log(this.message);
  while (true) {
  	console.log(this.message);
  }
  	this.reversedMessage = val + ' --- ';
  }
},
```



## 计算属性的setter

​		默认计算属性只有 getter，不过在需要时你也可以提供一个 setter。

```
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
```

​		在运行 `vm.fullName = 'John Doe'` 时，setter 会被调用，`vm.firstName` 和 `vm.lastName` 也会相应地被更新。

​		当然，如果你这样写，只能说你是小机灵鬼了，一直调用了 setter 方法导致溢出。

```
set: function (newValue) {
  console.log(newValue);
  this.reversedMessage += '1';
}
```

​		同时我们也可以使用一些简单的方式查看这个 setter 的执行时机。通过下面这个方式，我们发现了，setter 的执行在值的变化之前。

```
computed: {
  // 计算属性的 getter
  reversedMessage: {
    get: function () {
      // `this` 指向 vm 实例
      console.log(1);
      return this.message.split('').reverse().join('')
    },
    set: function (newValue) {
      console.log(this.reversedMessage, newValue);
      while (true) {
      console.log(this.reversedMessage);
    };
  }
},
```



## 侦听器

> ​		虽然计算属性在大多数情况下更合适，但有时也需要一个自定义的侦听器。这就是为什么 Vue 通过 `watch` 选项提供了一个更通用的方法，来响应数据的变化。当需要在数据变化时执行异步或开销较大的操作时，这个方式是最有用的。

​		简单来说就是对于一个异步，和一个开销大的操作时，监听器比较合适。

​		异步操作，限制访问频率(防抖)，设置中间状态等。



# Class 与 Style 绑定

```
https://cn.vuejs.org/v2/guide/class-and-style.html
```

​		在将 `v-bind` 用于 `class` 和 `style` 时，Vue.js 做了专门的增强。表达式结果的类型除了字符串之外，还可以是对象或数组。



## 绑定 HTML Class

### 对象语法

```
v-bind:class="{ active: isActive, 'text-danger': hasError }"
```

​		这样，就会根据 后面的真值来判断前面这个类是否能存在。并且这里 active 是一个字符串，就算这个 active 和后面的一个 data 数据重名，最终渲染的还是一个字符串。如何能让 active 也变成一个变量类型，

```
<div v-bind:class="{ [message]: flag }"></div>
```

​		这里使用了动态绑定，所以 message  会从data里面进行查找。找不到则为 undefined 的字符串。并且可以使用 .undefined 来进行操作。对于不是字符串的，会转为字符串处理。

​		并且绑定的数据对象不必内联定义在模板里。

```
<div v-bind:class="classObject"></div>

data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```

​		如果写在 data 里面，我还不知道如何将类名动态绑定。并且后面的 true 和 false 都是写死的那种，只能在后续使用方法改变。

```
    computed: {
      classObject: function () {
        return {
          [this.message]: this.flag
        }
      }
    },
```

​		如果写在计算属性里面，那么类名和真值都可以通过 this 进行获取。类名还是一样通过 [] 获取。不加上就会直接当成一个字符串。



### 数组语法

​		我们可以把一个数组传给 `v-bind:class`，以应用一个 class 列表

```
<div v-bind:class="[activeClass, errorClass]"></div>

data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

​		对于不是字符串的，不会被显示，需要是字符串类型才会显示。

​		同时，也能写三元表达式

```
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```

​		在数组语法中也可以使用对象语法：

```
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```



### 在组件上

​		当在一个自定义组件上使用 `class` property 时，这些 class 将被添加到该组件的根元素上面。这个元素上已经存在的 class 不会被覆盖。

```
Vue.component('my-component', {
  template: '<p class="foo bar">Hi</p>'
})

<my-component class="baz boo"></my-component>

<p class="foo bar baz boo">Hi</p>
```

​		在渲染的时候，重复的类名不会被消除。当然，最终的渲染结果还是看CSS的权重级别。



## 绑定内联样式

### 对象语法

​		`v-bind:style` 的对象语法十分直观——看着非常像 CSS，但其实是一个 JavaScript 对象。CSS property 名可以用驼峰式 (camelCase) 或短横线分隔 (kebab-case，记得用引号括起来) 来命名

```
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>

<div v-bind:style="{ color: activeColor, 'font-size': fontSize + 'px' }"></div>

data: {
  activeColor: 'red',
  fontSize: 30
}
```

​		当然也能绑定到一个样式对象。大致还是和上面的要求一样。



### 数组语法

​		`v-bind:style` 的数组语法可以将多个样式对象应用到同一个元素上：

```
<div v-bind:style="[activeColor]">123</div>

activeColor: {
	fontSize: '30px'
},
```

​		当然，也能将其作为一个数组加对象整合为一个返回值，绑定到一个样式对象。



### 自动添加前缀

​		当 `v-bind:style` 使用需要添加[浏览器引擎前缀](https://developer.mozilla.org/zh-CN/docs/Glossary/Vendor_Prefix)的 CSS property 时，如 `transform`，Vue.js 会自动侦测并添加相应的前缀。

​		其次，对于使用了 v-bind:style 和 style 的，会以 v-bind:style 为主。



### 多重值 （2.3.0）

​		从 2.3.0 起你可以为 `style` 绑定中的 property 提供一个包含多个值的数组，常用于提供多个带前缀的值

```
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

​		这样写只会渲染数组中最后一个被浏览器支持的值。在本例中，如果浏览器支持不带浏览器前缀的 flexbox，那么就只会渲染 `display: flex`。

​		**意思就是说**，对于这样的一个值的数组，我们会从后往前进行赋值，直到遇到浏览器可以支持的值，例如本例来说，先判断，flex，再判断 -ms-flexbox，最后再判断 -webkit-box。



# 条件渲染

```
https://cn.vuejs.org/v2/guide/conditional.html
```



## v-if

​		`v-if` 指令用于条件性地渲染一块内容。这块内容只会在指令的表达式返回 truthy 值的时候被渲染。

​		也可以用 `v-else` 添加一个“else 块”：

```
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>
```



### 在 `<template>` 元素上使用 `v-if` 条件渲染分组

​		因为 `v-if` 是一个指令，所以必须将它添加到一个元素上。但是如果想切换多个元素呢？此时可以把一个 `<template>` 元素当做不可见的包裹元素，并在上面使用 `v-if`。最终的渲染结果将不包含 `<template>` 元素。

```
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

​		这里很好理解。首先 v-if 只能添加到一个元素上，所以我们使用了一个元素进行了包裹，然后这个 template 的一个特点就是不会显示，例如

```
<template>
	<div>123</div>
</template>

最终的显示结果就是
<div>123</div>
```

​		所以这个的好处在于，既能产生包裹，还能不将其 DOM 的结构进行变化。



### v-else

​		你可以使用 `v-else` 指令来表示 `v-if` 的“else 块”

​		`v-else` 元素必须紧跟在带 `v-if` 或者 `v-else-if` 的元素的后面，否则它将不会被识别。

```
<div v-if="Math.random() > 0.5">
  Now you see me
</div>
<div v-else>
  Now you don't
</div>
```



### v-else-if（2.1.0）

​		`v-else-if`，顾名思义，充当 `v-if` 的“else-if 块”，可以连续使用

​		类似于 `v-else`，`v-else-if` 也必须紧跟在带 `v-if` 或者 `v-else-if` 的元素之后。

```
<div v-if="type === 'A'">A</div>
<div v-else-if="type === 'B'">B</div>
<div v-else>Not A/B/C</div>
```



### 用 `key` 管理可复用的元素

​		Vue 会尽可能高效地渲染元素，通常会复用已有元素而不是从头开始渲染。这么做除了使 Vue 变得非常快之外，还有其它一些好处。

```
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>
```

​		那么在上面的代码中切换 `loginType` 将不会清除用户已经输入的内容。因为两个模板使用了相同的元素，`<input>` 不会被替换掉——仅仅是替换了它的 `placeholder`。



​		对于上述的代码，切换了 input  的输入框，但是却对于 value 值没有发生改变。但是如果我们会发现对于类名，style等属性是会发生改变。我们同时也可以知道 value 是input输入框的值，如果是对于 DOM 元素，可以通过 value 进行获取，但是在这里，虽然使用了 value 的属性，但是只要进行输入了，value 的值也不起效果。

​		但是我们同时也发现了，再切换之后，DOM 的指向没有改变。并且也发现了 value 的值在控制台的输出是有变化的。只是对于输入框的内容没有变化。个人猜测，这里input的输入和value 其实中间不是完全直接对应。输入框显示的 value 只是作为了一个最初值。但是内部的value已经发生了变化。

​		当然解决这个方法很简单。

*  Vue 为你提供了一种方式来表达“这两个元素是完全独立的，不要复用它们”。只需添加一个具有唯一值的 `key` attribute 即可

  * ```
    <input placeholder="Enter your username" key="username-input">
    ```

  * 这里，只需要 key 值不同即可。并且发现加了 key 值之后， DOM 获取的元素已经不会根据你的按钮发生变化，并且值也不会发生变化了，因为已经不是一个 input 框了，就算是换回来也不是一个了，因为 v-if 是直接修改了 DOM 树。

  * ```
    <template v-if="loginType === 'username'">
      <label>Username</label>
      <input key="a" class="a" style="color: red; font-size: 20px" placeholder="Enter your username" value="1">
    </template>
    <template v-else>
      <label>Email</label>
      <input key="b" class="b" placeholder="Enter your email address" value="2">
    </template>
    
    
    记住，这个需要放在 vue实例之后，应该是因为 template 的原因。
    const a = document.querySelector('input.a'),
    	b = document.querySelector('input.b');
    function c() {
      console.log(a);
      console.log(a.value);
    }
    ```

* 当然，还可以使用 v-model 将数据进行绑定，那么input输入框的显示也会跟数据有关了。并且 v-model 是进行的复用。因为 DOM 的输出是会发生变化的。并且输入框和data数据是实时绑定了的。



## v-show

​		另一个用于根据条件展示元素的选项是 `v-show` 指令。用法大致一样

```
<h1 v-show="ok">Hello!</h1>
```

​		同的是带有 `v-show` 的元素始终会被渲染并保留在 DOM 中。`v-show` 只是简单地切换元素的 CSS property `display`。

> 注意，`v-show` 不支持 `<template>` 元素，也不支持 `v-else`。

​		v-show 不支持 template 元素，意思就是说，你在 template 元素上使用 v-show，不管是 true 还是 false，template 都会显示在页面上，而如果你使用v-if就会发现，结果是不一样的。

​		v-show 不支持 v-else，就如字面上来说，不支持的意思。



## v-if VS v-show

​		`v-if` 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。

​		`v-if` 也是**惰性的**：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

​		相比之下，`v-show` 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

​		一般来说，`v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 `v-show` 较好；如果在运行时条件很少改变，则使用 `v-if` 较好。

​		**简单概括：**

* **v-if** 是会直接和 DOM 树相关。而 **v-show** 只是简单的使用了 display:none，和渲染树相关。
* 所以 v-if 对于切换会产生高开销，因为每次都会进行 DOM 的修改。而 v-show 会产生初始渲染的高开销，因为不管是否显示都会渲染。
* 所以，对于频繁切换使用 v-show，对于很少改变使用 v-if。



## v-if 和 v-for 一起使用

> **不推荐**同时使用 `v-if` 和 `v-for`。请查阅[风格指南](https://cn.vuejs.org/v2/style-guide/#避免-v-if-和-v-for-用在一起-必要)以获取更多信息。

​		当 `v-if` 与 `v-for` 一起使用时，`v-for` 具有比 `v-if` 更高的优先级。请查阅[列表渲染指南](https://cn.vuejs.org/v2/guide/list.html#v-for-with-v-if)以获取详细信息。

​		当它们处于同一节点，`v-for` 的优先级比 `v-if` 更高，这意味着 `v-if` 将分别重复运行于每个 `v-for` 循环中。当你只想为*部分*项渲染节点时，这种优先级的机制会十分有用。

​		通过查看了 风格指南，主要说几点：

* 避免一起使用

* 对于需要过滤一个列表中的项目，采用计算属性

  * ```
    v-for="user in users" v-if="user.isActive"
    
    可以对 user 使用一个计算属性 activeUser 使用filter过滤后返回
    v-for="user in activeUsers"
    
    computed: {
    	activeUsers: function() {
    		return this.users.filter((user) => user.isActive);
    	}
    }
    ```

* 对于会直接应该被隐藏的列表，将 v-if 放在上层，不要在每次循环的时候判断

  * ```
    v-for="user in users" v-if="shouldShowUsers"
    
    shouldShowUsers 这是对一个 users 进行的判断，只要为 false，所有的 users都不会显示，所以这个建议放在上层
    v-if="shouldShowUsers"
    	v-for="user in users"
    ```



# 列表渲染

```
https://cn.vuejs.org/v2/guide/list.html
```



## 用 v-for 把一个数组对应为一组元素

​		我们可以用 `v-for` 指令基于一个数组来渲染一个列表。`v-for` 指令需要使用 `item in items` 形式的特殊语法，其中 `items` 是源数据数组，而 `item` 则是被迭代的数组元素的**别名**。

```
<ul id="example-1">
  <li v-for="item in items" :key="item.message">
    {{ item.message }}
  </li>
</ul>

items: [
  { message: 'Foo' },
  { message: 'Bar' }
]
```

​		`v-for` 还支持一个可选的第二个参数，即当前项的索引。

```
<li v-for="(item, index) in items">
	{{ parentMessage }} - {{ index }} - {{ item.message }}
</li>
```

​		index 从0开始。

​		你也可以用 `of` 替代 `in` 作为分隔符，因为它更接近 JavaScript 迭代器的语法

```
<div v-for="item of items"></div>
```



## 在 `v-for` 里使用对象

​		你也可以用 `v-for` 来遍历一个对象的 property。

```
<li v-for="value in object">
	{{ value }}
</li>

object: {
  title: 'How to do lists in Vue',
  author: 'Jane Doe',
  publishedAt: '2016-04-10'
}
```

​		你也可以提供第二个的参数为 property 名称 (也就是键名)

```
<div v-for="(value, name) in object">
  {{ name }}: {{ value }}
</div>

title: How to do lists in Vue
```

​		还可以用第三个参数作为索引

```
<div v-for="(value, name, index) in object">

0
1
2
```

> ​		在遍历对象时，会按 `Object.keys()` 的结果遍历，但是**不能**保证它的结果在不同的 JavaScript 引擎下都一致。



**注意：**

* 如果 v-for 里面是一个正整数n，那么将会变成 1~n，如果是一个小数，会报错。
  * 因为这个方法是会对其进行length操作，所以对于非正整数将出问题。
* 如果v-for里面是一个字符串，那么将会把字符串挨个字符输出。



## 维护状态

​		当 Vue 正在更新使用 `v-for` 渲染的元素列表时，它默认使用“就地更新”的策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素，并且确保它们在每个索引位置正确渲染。这个类似 Vue 1.x 的 `track-by="$index"`。

​		**简单来说就是**，发现了变化，不会查看是否是有匹配的 DOM，而是直接将原来位置上的DOM进行改变。比如如果只是位置发生了改变，如果使用默认的方式，那么就会挨着将DOM进行修改，但是如果使用了key来进行维护，那么会查看是否有key值存在的，有的话就会直接使用key的DOM进行维护。没有再创建。

​		这个默认的模式是高效的，但是**只适用于不依赖子组件状态或临时 DOM 状态 (例如：表单输入值) 的列表渲染输出**。

​		**这里给的意思就是说**，如果对于依赖了子组件话，那么进行修改的时候需要耗费大量的时间，所以需要进行一些判断来处理要使用什么方法。

​		为了给 Vue 一个提示，以便它能跟踪每个节点的身份，从而重用和重新排序现有元素，你需要为每项提供一个唯一 `key` attribute

​		就是说可以使用key来进行定位。

```
<div v-for="item in items" v-bind:key="item.id">
  <!-- 内容 -->
</div>
```

​		建议尽可能在使用 `v-for` 时提供 `key` attribute，除非遍历输出的 DOM 内容非常简单，或者是刻意依赖默认行为以获取性能上的提升。

​		因为它是 Vue 识别节点的一个通用机制，`key` 并不仅与 `v-for` 特别关联。后面我们将在指南中看到，它还具有其它用途。

​		其次对于key值，不要使用index下标进行赋值，因为如果你对数组进行了变化，位置变化等，可能下标也会发生改变，这样可能还会降低性能。

> ​		不要使用对象或数组之类的非基本类型值作为 `v-for` 的 `key`。请用字符串或数值类型的值。



## 数组更新检测

### 变更方法

​		因为Vue的响应式是相对于Object.defineProperty的使用。所以Vue对数组的处理方式是，对方法进行了包裹，所以使用了数组的方法也会触发视图的更新。

​		这些方法包括了

```
push，pop，shift，unshift，splice，sort，reverse
```



### 替换数组

​		变更方法，顾名思义，会变更调用了这些方法的原始数组。相比之下，也有非变更方法，例如 `filter()`、`concat()` 和 `slice()`。它们不会变更原始数组，而**总是返回一个新数组**。当使用非变更方法时，可以用新数组替换旧数组。

```
example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})
```

​		简单来说，上面的变更方法在调用之后是会变化原数组的。但是我们也有不会变更原数组的方法。所以对于这些不会变更原数组的方法，我们可以选择直接进行重新赋值。

```
items = newItems
```

​		我们可以发现对数组直接进行赋值也触发了视图的变化。因为我们对items这个数组也进行了监听。地址的改变也触发了视图的变化，同理，对于一个对象也是一样的。

​		你可能认为这将导致 Vue 丢弃现有 DOM 并重新渲染整个列表。幸运的是，事实并非如此。Vue 为了使得 DOM 元素得到最大范围的重用而实现了一些智能的启发式方法，所以用一个含有相同元素的数组去替换原来的数组是非常高效的操作。

> ​		由于 JavaScript 的限制，Vue **不能检测**数组和对象的变化。[深入响应式原理](https://cn.vuejs.org/v2/guide/reactivity.html#检测变化的注意事项)中有相关的讨论。



### 显示过滤/排序后的结果

​		有时，我们想要显示一个数组经过过滤或排序后的版本，而不实际变更或重置原始数据。在这种情况下，可以创建一个计算属性，来返回过滤或排序后的数组。

```
<li v-for="n in evenNumbers">{{ n }}</li>

computed: {
  evenNumbers: function () {
    return this.numbers.filter(function (number) {
      return number % 2 === 0
    })
	}
}
```

​		对于计算属性不适合的情况下，比如是循环嵌套了循环，此时对于循环内层，用不了计算属性，可以使用方法

```
<ul v-for="set in sets">
  <li v-for="n in even(set)">{{ n }}</li>
</ul>

even: function (numbers) {
  return numbers.filter(function (number) {
  	return number % 2 === 0
  })
}
```

​		当然，你可能会想着，我对这个内层也加一个计算属性啊，但是，是没有效果的，简单来说就是因为就近原则，一个是循环的set，一个计算属性的set，他会先找循环的set。



### 在 v-for 里使用值范围

​		`v-for` 也可以接受整数。在这种情况下，它会把模板重复对应次数。

​		对于字符串则会将字符进行循环。



### 在 \<template> 上使用 v-for

​		类似于 `v-if`，你也可以利用带有 `v-for` 的 `<template>` 来循环渲染一段包含多个元素的内容。



### 在组件上使用 `v-for`

​		在自定义组件上，你可以像在任何普通元素上一样使用 `v-for`。

```
<my-component v-for="item in items" :key="item.id"></my-component>
```

​		**2.2.0+ 的版本里，当在组件上使用 `v-for` 时，`key` 现在是必须的。**

​		然而，任何数据都不会被自动传递到组件里，因为组件有自己独立的作用域。为了把迭代数据传递到组件里，我们要使用 prop。

```
<ul>
  <li
    is="todo-item"
    v-for="(todo, index) in todos"
    v-bind:key="todo.id"
    v-bind:title="todo.title"
    v-on:remove="todos.splice(index, 1)"
  ></li>
</ul>

Vue.component('todo-item', {
  template: '\
    <li>\
      {{ title }}\
      <button v-on:click="$emit(\'remove\')">Remove</button>\
    </li>\
  ',
  props: ['title']
})
```

> ​		注意这里的 `is="todo-item"` attribute。这种做法在使用 DOM 模板时是十分必要的，因为在 `<ul>` 元素内只有 `<li>` 元素会被看作有效内容。这样做实现的效果与 `<todo-item>` 相同，但是可以避开一些潜在的浏览器解析错误。查看 [DOM 模板解析说明](https://cn.vuejs.org/v2/guide/components.html#解析-DOM-模板时的注意事项) 来了解更多信息。

​		简单来说，ul 元素内只有li元素被看作有效，我们使用is方法进行了替换，这样重点可以避开潜在的浏览器解析错误。当然这是一个Vue的方法。



# 事件处理

```
https://cn.vuejs.org/v2/guide/events.html
```



## 监听事件

​		可以用 `v-on` 指令监听 DOM 事件，并在触发时运行一些 JavaScript 代码。对应的语法糖，`@`

```
<button v-on:click="counter += 1">Add 1</button>
```



## 事件处理方法

​		然而许多事件处理逻辑会更为复杂，所以直接把 JavaScript 代码写在 `v-on` 指令中是不可行的。因此 `v-on` 还可以接收一个需要调用的方法名称。

```
<!-- `greet` 是在下面定义的方法名 -->
<button v-on:click="greet">Greet</button>

// 在 `methods` 对象中定义方法
methods: {
  greet: function (event) {
    // `this` 在方法里指向当前 Vue 实例
    alert('Hello ' + this.name + '!')
    // `event` 是原生 DOM 事件
    if (event) {
    	alert(event.target.tagName)
    }
  }
}

// 也可以用 JavaScript 直接调用方法
vm.greet() // => 'Hello Vue.js!'
```



## 内联处理器中的方法

​		除了直接绑定到一个方法，也可以在内联 JavaScript 语句中调用方法：

```
<button v-on:click="say('hi')">Say hi</button>

methods: {
  say: function (message) {
  	alert(message)
  }
}
```

​		有时也需要在内联语句处理器中访问原始的 DOM 事件。可以用特殊变量 `$event` 把它传入方法

```
<button v-on:click="warn('Form cannot be submitted yet.', $event)">Submit</button>
```



**对于event方法的使用：**

* 如果方法是不带参数的，可以直接使用event，或者参数上加上event

  * ```
    <button @click="change">button</button>
    
    change(event) {console.log(event);},
    change() {console.log(event);}
    ```

  * 不同点：

    * 如果使用了 @click=change()，加上了括号，对于第一个，传参event的，无法使用，第二个可以使用。

* 当然我们也可以使用 $event 来传递这个参数。



## 事件修饰符

​		在事件处理程序中调用 `event.preventDefault()` 或 `event.stopPropagation()` 是非常常见的需求。尽管我们可以在方法中轻松实现这点，但更好的方式是：方法只有纯粹的数据逻辑，而不是去处理 DOM 事件细节。

​		首先默认使用的click方法就是冒泡类型。

​		常见的事件修饰符

```
.stop
	阻止事件的冒泡
.prevent
	阻止事件的默认行为，对于在父元素使用了阻止默认行为，子元素的默认行为都会被阻止。
.capture
	转为捕获事件监听，事件的监听顺序是 root --> target 捕获， target --> root 冒泡
.self
	只有目标元素是自身才会触发，对于子元素的点击也不会触发。
.once	/ 2.1.4 新增
	只触发一次。
.passive / 2.3.0 新增
	
```

​		不像其它只能对原生的 DOM 事件起作用的修饰符，`.once` 修饰符还能被用到自定义的[组件事件](https://cn.vuejs.org/v2/guide/components-custom-events.html)上

> ​		使用修饰符时，顺序很重要；相应的代码会以同样的顺序产生。因此，用 `v-on:click.prevent.self` 会阻止**所有的点击**，而 `v-on:click.self.prevent` 只会阻止对元素自身的点击。

```
<div @click.prevent.self="change">
  <a href="#1" @click="change1">321</a>
  <div @click="change1">123</div>
</div>

<div @click.self.prevent="change">
  <a href="#1" @click="change1">321</a>
  <div @click="change1">123</div>
</div>
```

​		**热知识**：父元素阻止了默认行为会影响到子元素。

​		**热知识2：** click方法会先于默认行为执行。并且要冒泡结束了之后才会执行。

​		Vue 还对应 [`addEventListener` 中的 `passive` 选项](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Parameters)提供了 `.passive` 修饰符。

```
<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
<!-- 而不会等待 `onScroll` 完成  -->
<!-- 这其中包含 `event.preventDefault()` 的情况 -->
<div v-on:scroll.passive="onScroll">...</div>
```

​		他这里是这样说，貌似意思是说，默认行为会先触发，然后再触发 onScroll 的方法，但是我对一个。a标签进行操作的时候发现是先输出，然后在跳转，对于一个scroll行为的测试从肉眼上看也是和a标签一样，当然这个滚动的行为可能才滚1帧就开始触发了循环，导致卡帧也有可能。所以我现在不知道如何判断。

```
<div @click="change">
	<a href="#1" @click.passive="change1">321</a>
</div>

methods: {
  change() {
    console.log('father');
    let date = new Date().getTime() + 1000;
    while (date > new Date()) {

    };
  },
  change1() {
  	console.log('children');
  }
}
```

​		发现是先输出控制台，然后url再变化的。

​		并且如果父元素使用了 prevent，子元素的passive无效，passive只能让本元素上的prevent无效。

​		这个 `.passive` 修饰符尤其能够提升移动端的性能。 

​		

## 按键修饰符

​		在监听键盘事件时，我们经常需要检查详细的按键。Vue 允许为 `v-on` 在监听键盘事件时添加按键修饰符

```
<!-- 只有在 `key` 是 `Enter` 时调用 `vm.submit()` -->
<input v-on:keyup.enter="submit">
```



### 按键码

> ​		`keyCode` 的事件用法[已经被废弃了](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/keyCode)并可能不会被最新的浏览器支持。

​		使用 `keyCode` attribute 也是允许的：

```
<input v-on:keyup.13="submit">
```

​		为了在必要的情况下支持旧浏览器，Vue 提供了绝大多数常用的按键码的别名：

```
.enter	.tab	.delete (捕获“删除”和“退格”键)	.esc	.space	.up	.down	.left	.right
```

> ​		有一些按键 (`.esc` 以及所有的方向键) 在 IE9 中有不同的 `key` 值, 如果你想支持 IE9，这些内置的别名应该是首选。

​		你还可以通过全局 `config.keyCodes` 对象[自定义按键修饰符别名](https://cn.vuejs.org/v2/api/#keyCodes)：

```
// 可以使用 `v-on:keyup.f1`
Vue.config.keyCodes.f1 = 112
```

​		按键别名可以进行覆盖，当然这个不建议这样写已经存在的。

**注意：**

​		按键别名不要使用大写，因为大写的在 `<input v-on:keyup.enter="submit"> `，在这里会转为小写，所以无法使用成功。

```
<input type="text" @keyup.A="change">

Vue.config.keyCodes.A = 97;
```



## 系统修饰键

​		可以用如下修饰符来实现仅在按下相应按键时才触发鼠标或键盘事件的监听器。(2.1.0新增)

```
.ctrl
.alt
.shift
.meta
	就是Windows键盘上的那个Windows图标按钮。mac同理
```

> ​		注意：在 Mac 系统键盘上，meta 对应 command 键 (⌘)。在 Windows 系统键盘 meta 对应 Windows 徽标键 (⊞)。在 Sun 操作系统键盘上，meta 对应实心宝石键 (◆)。在其他特定键盘上，尤其在 MIT 和 Lisp 机器的键盘、以及其后继产品，比如 Knight 键盘、space-cadet 键盘，meta 被标记为“META”。在 Symbolics 键盘上，meta 被标记为“META”或者“Meta”。
>

```
<!-- Alt + C -->
<input v-on:keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div v-on:click.ctrl="doSomething">Do something</div>
```

**注意：**

* 使用系统修饰键对于 @keyup.67.ctrl 和 @keyup.ctrl.67 是一样的。不会有先后顺序。
* 请注意修饰键与常规按键不同，在和 `keyup` 事件一起用时，事件触发时修饰键必须处于按下状态。换句话说，只有在按住 `ctrl` 的情况下释放其它按键，才能触发 `keyup.ctrl`。而单单释放 `ctrl` 也不会触发事件。如果你想要这样的行为，请为 `ctrl` 换用 `keyCode`：`keyup.17`。17代表了ctrl
* **@keyup.17.67** 这个代表了按这两个其中一个都有效
* 系统修饰键可以使用多个。



### .exact 修饰符（2.5.0新增）

​		`.exact` 修饰符允许你控制由精确的系统修饰键组合触发的事件。

​		**作用**：用于**精确**控制系统修饰键按键的修饰符。主要在于精确两个字。并且是对系统修饰键起作用的。

```
<input type="text" @keyup.a.up.exact="change">
	这个里面没有系统修饰键，监听了两个按键，最终效果：没有什么区别，唯一的区别就是如果此时你按了系统修饰键将不会触发。
	所以这个代表了<!-- 没有任何系统修饰键被按下的时候才触发 -->
```

```
<input type="text" @keyup.ctrl.up.exact="change">
	这个里面存在了系统修饰键 ctrl，所以效果就是必须按了 ctrl才会有用，（当然这个是系统修饰键的效果），.exact修饰符 的效果就是，系统修饰键必须只按了ctrl才有用。精确。加上系统修饰键。
	其次.exact 修饰符没有位置的要求，和系统修饰键一样没有位置要求，
	然后就是对于 <input type="text" @keyup.exact.ctrl.up.a="change"> 我们可以发现， 一个exact修饰符，一个ctrl系统修饰键，两个普通按键修饰符。所以最终的效果是，有且只有按了ctrl键，加上普通按键修饰符的其中一个就行。
	<!-- 有且只有 Ctrl 被按下的时候才触发 -->
```



### 鼠标按钮修饰符（2.2.0新增）

​		这些修饰符会限制处理函数仅响应特定的鼠标按钮。

```
.left
.right
.middle
```

​		用于点击事件，对于keyup事件不起作用，当然对于系统修饰键和.exact修饰符没有这些要求。

```
<div @click.middle.ctrl.exact="change">321</div>
	要求是 ctrl键 + 鼠标中键才会触发。
```



## 为什么要在 HTML 中监听事件

​		你可能注意到这种事件监听的方式违背了关注点分离 (separation of concern) 这个长期以来的优良传统。但不必担心，因为所有的 Vue.js 事件处理方法和表达式都严格绑定在当前视图的 ViewModel 上，它不会导致任何维护上的困难。实际上，使用 `v-on` 有几个好处：

1. 扫一眼 HTML 模板便能轻松定位在 JavaScript 代码里对应的方法。
2. 因为你无须在 JavaScript 里手动绑定事件，你的 ViewModel 代码可以是非常纯粹的逻辑，和 DOM 完全解耦，更易于测试。
3. 当一个 ViewModel 被销毁时，所有的事件处理器都会自动被删除。你无须担心如何清理它们。



**概括就是说**：虽然是在html中进行的使用监听，但是真正的处理是绑定在VM上的。其次对于v-on的好处：1.能够一眼看出方法。2.和DOM完全解耦。3.当一个VM被销毁时，所有的事件会自动清除。



# 表单输入绑定

```
https://cn.vuejs.org/v2/guide/forms.html
```



## 基础用法

​		你可以用 `v-model` 指令在表单 `<input>`、`<textarea>` 及 `<select>` 元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。

```
<input v-model="message" placeholder="edit me">
这里，我是用 v-model，并没有绑定给value，但是会自动选取正确的方法进行更新。
```



**注意：**

* v-model 会忽略元素自带的value，checked等属性，而是使用Vue实例的数据作为来源。



​		`v-model` 在内部为不同的输入元素使用不同的 property 并抛出不同的事件。

- text 和 textarea 元素使用 `value` property 和 `input` 事件；
- checkbox 和 radio 使用 `checked` property 和 `change` 事件；（使用的是真值方式truth）
- select 字段将 `value` 作为 prop 并将 `change` 作为事件。



> ​		对于需要使用[输入法](https://zh.wikipedia.org/wiki/输入法) (如中文、日文、韩文等) 的语言，你会发现 `v-model` 不会在输入法组合文字过程中得到更新。如果你也想处理这个过程，请使用 `input` 事件。

​		你在输入框输入加上一个input事件的监听的时候就会发现。如果在输入的时候使用了中文，虽然在按键的时候发生了input事件，但是v-model的值并没有得到更新。

​		但是如果是普通的input输入框的监听则会发生更新。

```
<input type="text" v-model="string" @input="change" oninput="console.log('event:    ' + event.target.value);">
```

​		并且在进行了空格之后会发生多次的更新。



### 文本 和 多行文本

```
<input type="text" v-model="message">
<textarea v-model="message"></textarea>
```

​		在文本区域插值 (`<textarea>{{text}}</textarea>`) 并不会生效，应用 `v-model` 来代替。



### 复选框

​		单个复选框，直接布尔值进行的判断，对于不是布尔类型的使用了truth方式。

```
<input type="checkbox" id="checkbox" v-model="checked">
```

​		对于多个复选框

​		注意：复选框和单选框是通过value进行判断。

```
<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
<label for="jack">Jack</label>
<input type="checkbox" id="john" value="John" v-model="checkedNames">
<label for="john">John</label>
<input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
<label for="mike">Mike</label>
```

​		~~我们可以发现，这里没有对复选框进行分组，正常的情况下，复选框需要进行name的分组，相同的name为一组。这个好像是对单选框的。复选框应该本来就可以不用分组？~~

​		对于一个复选框，如果绑定了v-model，但是value不绑定，那么点击一个就是点击多个。因为没有使用value属性，值为null，所以所有为null都会被同步变化。

​		同时，如果多选的复选框，但是绑定的属性不是一个数组那么最终也会变为全部都会出现相同的变化。 `checkedNames: 1,` 





### 单选按钮

```
  <input type="radio" name="aa" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="radio" name="cc" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="radio" name="aa" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
```

​		在这里，我将name进行不同的划分，但是可以发现他们还是一组的成员。

​		~~所以我们可以这样认为，使用了v-model之后，name也会绑定为这个属性的名称，所以你自己定义的属性名称是没有作用的。~~（注意：这里不是说，绑定的是v-model属性的名称，而是说，name的绑定和v-model的属性相关了。但是值不知道是什么。）

​		**注意：** 首先我们可以测试出来，name的属性还是没有改变，因为如果添加了一个 相同name，但是没有使用v-model的，会出现竞争。

​		对于单选按钮，**没有使用value的**，那么v-model绑定的属性取出来的值是空。就是那种什么都没有的空。**并且name属性默认不同**。

​		如果自己定义了name属性，那么会以自己定义的为准。但是如果使用了value，搭配了v-model，对于同value，不同name，两个都选上。 ~~那么name属性还是以v-model为准（是指相同的v-model有相同的name）。~~

​		并且，如果value相等，name不等，那么点击时，都会一起变化。如果name相等了，那么点击时点击那个就是哪个，但是value的值不变，并且对于初始化来说，是根据value的值来进行的变化，所以会以最后一个为准。

​		这里有很多问题，但是这些都是可以手动避免的。大概知道就行。我也被自己扯蒙了。



### 选择框

​		单选时，直接绑定一个值即可

```
  <select v-model="selected">
    <option disabled value="">请选择</option>
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <span>Selected: {{ selected }}</span>
```

​		我们也可以发现，对于使用option的时候，可以不添加value属性，此时绑定的值就是内容。

**注意：**如果没有规定 value 属性，选项的值将设置为 \<option> 标签中的内容。

**注意：**

> ​		如果 `v-model` 表达式的初始值未能匹配任何选项，`<select>` 元素将被渲染为“未选中”状态。在 iOS 中，这会使用户无法选择第一个选项。因为这样的情况下，iOS 不会触发 change 事件。因此，更推荐像上面这样提供一个值为空的禁用选项。

​		当然这个我不清楚，毕竟我没有IOS。。。。。。



​		多选时就是绑定一个数组

```
<select v-model="selected" multiple style="width: 50px;">
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<br>
<span>Selected: {{ selected }}</span>
```

​		首先，select多选框的属性 multiple，其次就是使用的数组了。

​		对于不是使用数组的，将不会初始化成功，但是在后续的赋值还是会转化为数组。

​		当然，对于 option 也可以使用v-for进行动态渲染。

```
<option v-for="item in options">{{ item }}</option>
```



## 值绑定

​		对于单选按钮，复选框及选择框的选项，`v-model` 绑定的值通常是静态字符串 (对于复选框也可以是布尔值)：

​		但是有时我们可能想把值绑定到 Vue 实例的一个动态 property 上，这时可以用 `v-bind` 实现，并且这个 property 的值可以不是字符串。

```
<!-- 当选中时，`picked` 为字符串 "a" -->
<input type="radio" v-model="picked" value="a">
```

```
<!-- 当选中时，`picked` 为a 的值 -->
<input type="radio" v-model="picked" :value="a">
```



### 复选框

```
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>

// 当选中时
vm.toggle === 'yes'
// 当没有选中时
vm.toggle === 'no'
```

​		针对多个复选框。其值还是为value值，如果没有value，其值就是null

> ​		这里的 `true-value` 和 `false-value` attribute 并不会影响输入控件的 `value` attribute，因为浏览器在提交表单时并不会包含未被选中的复选框。如果要确保表单中这两个值中的一个能够被提交，(即“yes”或“no”)，请换用单选按钮。

​		所以这个 true/false value 是单选时比较有用。



### 单选按钮

```
<input type="radio" v-model="pick" v-bind:value="a">

// 当选中时
vm.pick === vm.a
```

​	

### 选择框的选项

```
<select v-model="selected">
    <!-- 内联对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>

// 当选中时
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

​		通过这个我们可以看出，这个是可以使用对象的，同理，我们对于其他的选项框也可以使用对象的形式。



## 修饰符

### .lazy

​		在默认情况下，`v-model` 在每次 `input` 事件触发后将输入框的值与数据进行同步 (除了[上述](https://cn.vuejs.org/v2/guide/forms.html#vmodel-ime-tip)输入法组合文字时)。你可以添加 `lazy` 修饰符，从而转为在 `change` 事件_之后_进行同步

```
<!-- 在“change”时而非“input”时更新 -->
<input v-model.lazy="msg">
```

​		在输入之后使用回车，便是change事件。



### .number

​		如果想自动将用户的输入值转为数值类型，可以给 `v-model` 添加 `number` 修饰符

```
<input v-model.number="age" type="number">
```

​		无法输入字符串。

​		这通常很有用，因为即使在 `type="number"` 时，HTML 输入元素的值也总会返回字符串。如果这个值无法被 `parseFloat()` 解析，则会返回原始的值。

​		如何出现无法解析的情况，因为可以输入 e，+，-等，所以还是可以无法解析，问题在于无法解析输出的类型是字符串，但是貌似内容为''，



### .trim

​		如果要自动过滤用户输入的首尾空白字符，可以给 `v-model` 添加 `trim` 修饰符：

```
<input v-model.trim="msg">
```

​		没啥说的。就是字符串的 trim 方法。这个方法的使用是返回一个新的。



## 在组件上使用 v-model （2.2.0+ 新增）

​		HTML 原生的输入元素类型并不总能满足需求。幸好，Vue 的组件系统允许你创建具有完全自定义行为且可复用的输入组件。这些输入组件甚至可以和 `v-model` 一起使用！

​		要了解更多，请参阅组件指南中的[自定义输入组件](https://cn.vuejs.org/v2/guide/components-custom-events.html#自定义组件的-v-model)。

​		讲真，没有看懂。

​		我们通过跳转，看到了自定义组件的 v-model 我只能大概知道

​		一个组件上的 `v-model` 默认会利用名为 `value` 的 prop 和名为 `input` 的事件，但是像单选框、复选框等类型的输入控件可能会将 `value` attribute 用于[不同的目的](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox#Value)。`model` 选项可以用来避免这样的冲突

```
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})

使用v-model时
<base-checkbox v-model="lovingVue"></base-checkbox>
```

​		这里因为使用了v-model进行传值，所以使用了model: {}，设置了prop的名字，然后在props进行使用，此时传递的值就和父组件的 lovingVue 进行了绑定。然后通过事件$emit， change进行的提交。

​		这里的 `lovingVue` 的值将会传入这个名为 `checked` 的 prop。同时当 `<base-checkbox>` 触发一个 `change` 事件并附带一个新的值的时候，这个 `lovingVue` 的 property 将会被更新。

> ​		注意你仍然需要在组件的 `props` 选项里声明 `checked` 这个 prop。



# 组件基础

```
https://cn.vuejs.org/v2/guide/components.html
```



## 基本示例

​		这里有一个 Vue 组件的示例

```
// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```

​		组件是可复用的 Vue 实例，且带有一个名字：在这个例子中是 `<button-counter>`。我们可以在一个通过 `new Vue` 创建的 Vue 根实例中，把这个组件作为自定义元素来使用

```
<div id="components-demo">
  <button-counter></button-counter>
</div>
```

​		当然注意上面这个组件的定义顺序要在你的Vue实例之前。因为编译问题，如果在之后的话就不会被编译了。

```
Vue.componnet('button', { });

const vm = new Vue();
```

​		因为组件是可复用的 Vue 实例，所以它们与 `new Vue` 接收相同的选项，例如 `data`、`computed`、`watch`、`methods` 以及生命周期钩子等。仅有的例外是像 `el` 这样根实例特有的选项。



## 组件的复用

​		组件在被创建之后，可以被多次使用。

### data必须是一个函数

​		当我们定义这个 `<button-counter>` 组件时，你可能会发现它的 `data` 并不是像这样直接提供一个对象

```
Vue.component('button', {
	data: {
		return {
			
		};
	},
})

const vm = new Vue({
	data: {
		
	},
})
```

​		当然如果你不这样写也不会报错，但是这样会有一个问题，那就是所以这个组件都会使用同一个对象的数据，一个发生了改变，所有都会发生改变。所以就是用了函数，这个会每次都调用了一次函数，形成一个新的作用域位置。

​		这个就是使用了闭包的方法，当然你可能在想，是不是可以利用这个闭包，然后既能让数据不同步，又能让部分数据进行同步。当然，我没有解决掉。因为首先我们可以知道闭包的使用方式。

```
function Fn() {
	return fn() {}
}

let fn = Fn();
这个时候使用fn，就可以使用闭包。但是data是重复的创建Fn(). 所以每次都还是会创建新东西。
```

​		所以我现在不知道如何使用闭包的方式，当然，我们可以将方法写在全局，然后再组件里进行闭包。还是可以的。



## 组件的组织

​		为了能在模板中使用，这些组件必须先注册以便 Vue 能够识别。这里有两种组件的注册类型：**全局注册**和**局部注册**。至此，我们的组件都只是通过 `Vue.component` 全局注册的

​		全局注册的组件可以用在其被注册之后的任何 (通过 `new Vue`) 新创建的 Vue 根实例，也包括其组件树中的所有子组件的模板中。

​		局部注册的方式就是先将内容对象赋值给了一个变量，然后通过使用变量来进行注册。

```
let ComponentA = {  }

new Vue({
	el: '#xx',
	components: {
		'component-a': ComponentA,
	},
})
```



## 通过Prop向子组件传递数据

​		简单来说，就是写在props属性里面的会添加到属性，父组件在调用子组件时，可以通过在标签里添加对应的属性，属性里面的值将会传递给子组件。

```
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})

<blog-post title="My journey with Vue"></blog-post>
```

​		当然对于这个props，也有另一个写法，props使用对象，对象里面又是一个对象，default代表了默认值，type代表了类型，当然也有其他属性。但是我们后面在详细了解。

```
props: {
  'title': {
    default: '123',
    type: String
  }
},
```

```
<blog-post title="My journey with Vue"></blog-post>
<blog-post></blog-post>
```

​		当然这个自定的属性attribute，也是可以使用v-bind: 来动态绑定。当然我们也可以使用v-model，但是这里也有其他的问题，具体后续在了解。

​		传递的属性也能是对象，对于一些应该属于统一对象的，可以将其化为一个对象进行传递。



## 单个根元素

​		简单来说，就是在创建时，只能以一个根元素。如果根元素不止一个就会报错

```
<div></div>
<div></div>
```

​		上面这个写法就会报错，但是下面这个写法。将所有的元素都放在了一个根元素的下级。

```
<div>
  <div></div>
  <div></div>
</div>
```

​		这里的原因。我不清楚，只能说在Vue里面如果使用了下面这个写法时，可以知道，只会将第一个进行Vue的渲染，第二个将不会进行渲染操作。通过查阅资料，有的说是diff算法的原因，也有说是为了避免出现多个根元素，找不到以谁为主体。

​		这个现在主要作为一个了解。

```
<div id="app">1</div>
<div id="app">2</div>
```



## 监听子组件事件

​		父组件可以给子组件传值了，但是子组件如何在一定的条件下通知父组件呢。

​		使用方式：

​		1.首先父元素在传递的时候，传递一个可以被子元素监听的方法

```
<blog-post @test="enlarge" post="{title: 1}"></blog-post>
```

​		2.然后子元素就可以通过使用 $emit 进行调用这个方法。注意$emit('xxxx')，xxx就是那个元素上的属性attribute，

```
<button v-on:click="$emit('test')">
```

​		3.传递值的方式，这个方法的第一个是方法名，后面的就是要传递的值。

```
$emit('test', 1, 2);
```

**注意：**

* 父元素进行传递时，直接写上方法名即可 `@test="enlarge"` 

* 因为html是不分大小写的，所以进行传递的时候，建议不要带有大写，对于$emit('xxx')，存在大写，则会监听失败。

* 如果是 `@test="enlarge()"` 那么子组件传递参数则无效，`$emit('test', 1, 2);` 子组件这个写法虽然传递了参数，但是并不会传递值，因为父组件在传递的时候是直接传递了方法的调用的结果。

  * ```
    @test="enlarge()
    $emit('test', 1, 2);
    
    结果：空
    ```

*  `@test="enlarge(1, 2)` 同理，这样在子组件调用的时候传递过来的值就直接是是1和2。并不会因为 `$emit('test', 321, 123);` 改成321，123。

  * ```
    @test="enlarge(1, 2)
    $emit('test', 11111, 22222);
    
    结果：1, 2
    ```

*  `@test="enlarge($event, 12, 321, 312)"` 这样写有是一个特点，你会发现，这个$event 代表的不是点击的事件了，而是子组件传递的值。这个event就类似于了一个子组件的待定参数。

  * ```
    @test="enlarge($event, 1, 2)
    $emit('test', 11111, 22222);
    
    结果：11111, 1, 2
    ```



### 使用事件抛出一个值

​		可以使用 `$emit` 的第二个参数来提供这个值

```
<button v-on:click="$emit('enlarge-text', 0.1)">
```

​		然后当在父级组件监听这个事件的时候，我们可以通过 `$event` 访问到被抛出的这个值

```
@enlarge-text="postFontSize += $event"
```

​		如果这个是一个方法，那么这个值会作为第一个参数传入这个方法

```
onEnlargeText: function (enlargeAmount) {
	this.postFontSize += enlargeAmount
}
```



### 在组件上使用 v-model

​		首先我们可以这样理解

```
<input v-model="searchText">

等价于

<input
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>
```

​		v-model的效果就是值的改变会影响到view的改变，输入的变化会影响值的改变。而v-bind，值的改变会影响到视图的改变，但是并没有双向的绑定。



​		所以用在组件上时。

```
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event"
></custom-input>
```

​		所以此时我们需要绑定一个input方法将其传递出来。

​		为了让它正常工作，这个组件内的 `<input>` 必须：

- 将其 `value` attribute 绑定到一个名叫 `value` 的 prop 上
- 在其 `input` 事件被触发时，将新的值通过自定义的 `input` 事件抛出

```
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >`
})
```

​		所以此时我们就能理解了，上面的那个组件使用v-model时的传递方式了。

​		同时我么可以看一下那个[自定义事件的 v-model](https://cn.vuejs.org/v2/guide/components-custom-events.html)

```
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >`
})

使用
<base-checkbox v-model="lovingVue"></base-checkbox>
```

​		这里的model里面有prop和event，其中checked代表了传递的值。这个名字可以自己定义。 这里的event，值为change，则代表了是change事件，如果命名为input则为input的事件，当然，其实这里也是可以自己命名的。主要是为了方便认知。

​		这个和上一个组件的通信的区别

* 普通的是将值返回给上级，然后上级进行方法的操作。

  * ```
    <c :name="name" @inputf="fn">12</c>
    
    子组件的内容：
    <input type="text" :value="name" @input="this.$emit('inputf', event.target.value);">
    
    这里再对fn写上一个方法，此时对于一个input输入就会出现对应的方法被执行。
    ```

* 对于v-model，则是发现直接将其传递给上级，上级不需要再指定一个方法。

  * ```
    <c v-model="searchText"></c>
    
    子组件的内容
    <input type="text" :value="myInput" @input="inputE($event)">
    ```

  * 首先在使用组件的时候，使用v-mdoel方法。子组件，此时可以使用 model对象进行指定。当然，如果此时不指定怎么办

  * 对于有value的情况：

    * 首先，对于父组件使用v-model传递给了子组件的值，子组件如果使用了value的变量名，则会以此值进行接收。就算是checkbox，也是使用的value进行接收。不管子组件的内容(目前我的测试来说。)

    * ```
      props: {
        value: {
        	type: Boolean,
        	defalut: false,
        },
      },
      
      记住props的写法，开始我写成了data式的写法，把默认值直接写在了后面，如果直接写后面是写变量的类型
      props: {
      	value: String,
      }
      ```

  * 对于没有value的情况：

    * 没有value，还没有添加一个model对象进行指定，那么就不会传入成功。

    * 进行了model的指定，那么便会使用这个变量进行赋值。

      * ```
        model: {
          prop: 'myInput',
          event: 'inp'
        },
        props: ['myInput'],
        ```

  * 到此，我们已经解决了如何传值，下一步就是更新数据。

    * 因为 v-model的特点就是会将值进行了绑定，所以我们只需要通知同步就行了

    * ```
      <input type="text" :value="myInput" @input="inputE($event)">
      ```

    * input事件，绑定了inputE方法，然后inputE里面通过$emit进行传递。事件名称就是model里面的事件名称，inp, 如果没有进行重命名，那么就是 input事件。父组件不需要做什么，因为v-model自动对事件和参数进行了赋值。当然，也是可以赋值常数的。

  * 同时我们通过这个案例也知道了，如果你对一个input输入框加了v-model，也加了input的监听，在input的监听修改了v-model的值，那么会以input的为主。



## 通过插槽分发内容

​		简单来说，就是可以在标签内部使用标签，然后标签可以传递给子元素显示。父元素的使用方式就是下面这样。子元素只需要定义一个 slot，然后slot的位置就会显示为你定义的。

```
<alert-box>
  Something bad happened.
</alert-box>
```

```
Vue.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot> 这里就会被渲染为其他的。
    </div>`
})
```

​		这里在简单的说几个地方。

​		1.如果使用了多个slot，默认每个slot都会全部都会显示，但是又不是你想的那种显示。我不知道怎么描述。直接看例子就懂了：

```
<p>123</p>
<slot></slot>
<slot></slot>
<p>321</p>


<tt>
  <div>123</div>
  <div>321</div>
</tt>
```

​		下面这个我定义了两个slot，然后组件名为tt，tt里面有两个div标签值为123和321。你可能会认为一个 slot为123，一个slot为321.但是其实不是，因为你没有给定name，所以这两个div会被当成一个传递给插槽。然后两个插槽都会被赋值。所以值为

<img src="Vue2-教程-基础使用/image-20211105095952701.png" alt="image-20211105095952701" style="zoom:50%;" />



​		那么要如何实现上面预想的效果呢，使用name。一个不带 `name` 的 `<slot>` 出口会带有隐含的名字“default”。

​		然后就是父组件如何使用了，这里有三个写法

```
<div slot="aa">123</div>

<template v-slot:aa>
	<div>123</div>
</template>

<template #aa>
	<div>123</div>
</template>
```

​		上面这三个写法，

* 第一个是一个旧的写法，不建议，因为建议是使用一个template进行包裹，template作为一个html5的新特性。

* 第二个是使用的v-slot进行绑定。但是需要将其放在template上，才有效果

* 第三个就是第二个的一个语法糖写法。

* 然后就是如何使用变量，首先可以使用 v-bind 进行绑定。其次也可以使用 [] 进行表示。

  * ```
    :slot="name" :v-slot:name :#name
    v-slot:[name] #[name]
    ```



​		然后就是插槽是会将值进行覆盖的。所以如果你在slot上写的一些样式和方法不会显示，对应的方式就是（**v-if，v-for** 有效果，因为这些是对DOM树进行了变化，所以会在DOM树的添加，而不是对一个属性的覆盖。但是对于一些class和v-show就没用了。）

* 第一种，父元素写方法和样式，但是这样一个子组件就对父组件不透明了

* 第二种，在外面套上一层标签。向下面这样就行了。

  * ```
    <div v-show="false">
    	<slot></slot>
    </div>
    ```



​		其他的部分，详见Vue的 [插槽](https://cn.vuejs.org/v2/guide/components-slots.html) 部分。



## 动态组件

​		简单来说就是不同组件会进行动态切换，所以可以使用

```
<component :is="name"></component>
<button @click="change">change</button>
```

​		这里 component 是一个标签，is使用v-bind绑定了name，然后通过一个点击事件来修改了name的值，所以component，is就会被指定修改成其他的组件名。

​		这个is属性应该是html里面的那个is属性，但是具体的使用方式我现在看不懂，可以取MDN里面进行了解，这里我们就当作is会指定一个组件名，然后这个component标签就会被替换成组件名。



​		在上述示例中，name可以包括：

- 已注册组件的名字，或
- 一个组件的选项对象



注意：

​		这个is属性可以用于常规的html元素上。

​		但是对于attribute将会作为DOM attribute进行绑定，对于像 `value` 这样的 property，若想让其如预期般工作，你需要使用 [`.prop` 修饰器](https://cn.vuejs.org/v2/api/#v-bind)。

​		这里就扯出了 attribute 和 property 的一个区别，我这里就贴一个 [StackOverflow](https://stackoverflow.com/questions/6003819/properties-and-attributes-in-html#answer-6004028) 和 一个对应 StackOverflow 的 [CSDN ](https://blog.csdn.net/rudy_zhou/article/details/104058741) 的一个中文的讲解 

​		大概就是说，attribute属性是一个HTML的上的属性，而property是一个DOM对象上的属性。有的属性开始是继承了attribute，但是后续会被修改，此时可以从property看出，但是不会从attribute看出。



## 解析 DOM 模板时的注意事项

​		简单来说，就是有的HTML 元素限制了其内部的元素是哪些，对于不属于的，会被提升到外部，触发其他问题。

```
<ul>、<ol>、<table> 和 <select>
```

​		有的元素是，只能存在于特定的元素内部

```
<li>、<tr> 和 <option>
```

​		案例

```
<table>
  <blog-post-row></blog-post-row>
</table>
```

​		对于上面的情况，blog-post-row 会被提升到外部，所以会出现页面布局的问题。

​		解决方式，使用is attribute。

```
<table>
  <tr is="blog-post-row"></tr>
</table>
```



需要注意的是**如果我们从以下来源使用模板的话，这条限制是*不存在* 的**：

- 字符串 (例如：`template: '...'`) 

  - ```
    Vue.component('tt', {
      template: `
        <table>
        	<p>1</p>
        </table>
      `,
    });
    ```

  - 此时 p 标签存在于table 内部。

- [单文件组件 (`.vue`)](https://cn.vuejs.org/v2/guide/single-file-components.html) 

- [`<script type="text/x-template">`](https://cn.vuejs.org/v2/guide/components-edge-cases.html#X-Template) 

  ​	当然这些，我也没有测试过，所以不清楚。



至此，简单的一个基础就了解完了，详细的还是看看Vue官网的 文档和API吧

```
https://cn.vuejs.org/v2/api/
```

```
https://cn.vuejs.org/v2/guide/
```

