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
  template: '<li>这是个待办项 {{ todo }}</li>',
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
<span>Message: {{ msg }}</span>
```

这里 msg 会替代为 数据对象的 msg。并且还带有响应式的功能。

​		通过使用 [v-once 指令](https://cn.vuejs.org/v2/api/#v-once)，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上的其它数据绑定：

```
<span v-once>这个将不会改变: {{ msg }}</span>
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
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

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
{{ message.split('').reverse().join('') }}
```

​		在模板中放入太多的逻辑会让模板过重且难以维护。

​		在这个地方，模板不再是简单的声明式逻辑。你必须看一段时间才能意识到，这里是想要显示变量 `message` 的翻转字符串。当你想要在模板中的多处包含此翻转字符串时，就会更加难以处理。

​		所以，对于任何复杂逻辑，你都应当使用**计算属性**。

​		我认为，从一个开发来看，对于一个表达式，如果以后会有多个地方进行相同的逻辑的使用，就应当使用计算属性，方便维护。



### 基础例子

```
<div id="app-6">
  {{ message }}
  <br>
  {{ reversedMessage }}
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
<p>{{ reversedMessage() }}</p>

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
  {{ message }}
  <br>
  {{ reversedMessage }}
  <br>
  {{ reversedMessage }}
  <br>
  {{ reversedMessage }}
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

