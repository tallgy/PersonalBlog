---
title: Vue-使用-深入了解组件
date: 2021-10-29 23:15:14
tags:
 - Vue
 - Vue2
 - 文档
categories:
 - Vue
 - Vue2文档
---



#  组件注册

```
https://cn.vuejs.org/v2/guide/components-registration.html
```



## 组件名

### 组件名大小写

​		对于大小写的组件名，在使用的时候大写会变成 -+小写的形式。

```
myComPonent

my-com-ponent
```



## 全局注册

```
Vue.component('component-a', { /* ... */ })
```

​		这样创建就是全局注册的，只要注册了之后，后面的Vue实例都可以直接使用。



## 局部注册

​		全局注册会不可避免的增加性能的消耗，浪费了很多时间。所以可以使用局部注册的方式进行注册，将组件注册在实例的内部，因此在实例被销毁时也会被销毁。

```
const ComponentA = { /* ... */ };

new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```

​		简单来说就是 ComponentsA 里面的对象就是 new一个Vue实例的对象。然后再在components里面进行new的创建，所以将作用域限制到了一定的范围。

```
const C = {
  template: `<div>22</div>`,
}
const ComponentA = {
  el: '#ap',
  //这里是局部组件创建再加上了一个组件的创建，否则一个父组件，两个局部组件内部是不能相互调用的。
  components: {
    'com': C,
  }
}

const app = new Vue({
  el: '#app',
  data: {
    a: 1
  },
  components: {
    'comA': ComponentA,
    'com': C,
  }
})
```



## 模块系统

​		简单来说就是可以使用 import / require 来使用一个模块系统。



### 在模块系统中局部注册

​		简单来说，下面这个是取出了一个对象。这个是一个ES6的模块化导出的默认导出的写法。

​		所以简单来说就是将对象进行导出。

```
import ComponentA from './Component';
```

```
export default {
	
}
```

​		我们先理清一下思路

* 首先，Vue的components里面使用的是一个对象，这个对象是那个实例对象。其次对于这个命名的思路是因为是es6的对象赋值的方式
* 然后就是导入 import 和 export default 这里导入和导出对象。
* 所以其实还是有思路的。



### 基础组件的自动化全局注册

​		require.context 可以全局注册组件。但是需要使用webpack或者使用了VueCLI3+（因为内部使用了webpack）

​		**什么是 require.context ：**

* 首先，是一个webpack的api。
* 其次，这个api用于实现自动化导入模块。就是对于一个文件引入很多模块的情况，可以使用这个api，会遍历指定的文件，然后进行自动导入，不需要每次显式的调用import导入模块。
* 进行一个更细粒度的模块引入。



​		**一个使用时机**：

* 首先就是需要引入很多模块
* 其次就是这个模块的处于同一父文件位置，所以对于基础组件来说是非常合适的。



​		require.context 的参数：

* directory，String类型

  * 文件目录位置，

* includeSubdirs，Boolean类型

  * 表示是否包含文件的子目录，可选参数，默认是 true

* filter，RegExp正则表达式类型

  * 表示过滤某些文件。可选参数，默认是 `/^\.\/.*$/` 指的是所有文件。（这里是webpack写的，但是我没有理解这个正则）。

* mode，String类型

  * 表示加载的方式， sync，eager，weak，lazy，lazy-once。默认值是sync。

  * ```
    sync
    eager
    	不会生成额外的chunk，所有模块当成当前chunk引入。没有额外的网络请求，但是会返回一个resolved的Promise。
    weak
    	这个没有看懂，大概可能是尝试加载，不可用返回一个reject的Promise。
    lazy
    	为每一个导入的模块生成一个可延迟加载的chunk。简单来说就是将以异步方式加载。
    lazy-once
    	生成一个可以满足所有的可延迟加载的chunk。这个chunk将第一次调用时获取，随后使用相同的网络响应。
    ```



大概看懂了之后，我们再看一下组件的自动化全局注册

```
import Vue from 'vue'
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'

const requireComponent = require.context(
  // 其组件目录的相对路径
  './components',
  // 是否查询其子目录
  false,
  // 匹配基础组件文件名的正则表达式
  /Base[A-Z]\w+\.(vue|js)$/
)

requireComponent.keys().forEach(fileName => {
  // 获取组件配置
  const componentConfig = requireComponent(fileName)

  // 获取组件的 PascalCase 命名
  const componentName = upperFirst(
    camelCase(
      // 获取和目录深度无关的文件名
      fileName
        .split('/')
        .pop()
        .replace(/\.\w+$/, '')
    )
  )

  // 全局注册组件
  Vue.component(
    componentName,
    // 如果这个组件选项是通过 `export default` 导出的，
    // 那么就会优先使用 `.default`，
    // 否则回退到使用模块的根。
    componentConfig.default || componentConfig
  )
})
```

​		**其中这一部分的作用是将每个文件的模块给取了出来**

```
const requireComponent = require.context(
  // 其组件目录的相对路径
  './components',
  // 是否查询其子目录
  false,
  // 匹配基础组件文件名的正则表达式
  /Base[A-Z]\w+\.(vue|js)$/
)

目录是 ./components
不查询子目录
匹配文件的形式，Basexxxx.(vue|js)
```

* requireComponent 通过typeof 判断是一个，function，里面存在了方法，可以使用keys方法进行获取



​		**这里是将requireComponent存储的组件给进行了注册。**

```
requireComponent.keys().forEach(fileName => {
  // 获取组件配置
  const componentConfig = requireComponent(fileName)

  // 获取组件的 PascalCase 命名
  const componentName = upperFirst(
    camelCase(
      // 获取和目录深度无关的文件名
      fileName
        .split('/')
        .pop()
        .replace(/\.\w+$/, '')
    )
  )

  // 全局注册组件
  Vue.component(
    componentName,
    // 如果这个组件选项是通过 `export default` 导出的，
    // 那么就会优先使用 `.default`，
    // 否则回退到使用模块的根。
    componentConfig.default || componentConfig
  )
})
```

* keys方法获取到的是文件的相对路径。使用foreach进行循环操作。foreach内部也是和forin一个意思。forin和forof的区别在于迭代器问题。

  * ```
    ./App.vue
    ./components/App.vue
    ```

* 然后使用 requireComponent 方法，参数为文件位置，便会获取到内容。

  * 使用 requireComponent(filename)，进行获取，就可以获取到内容。

* 当然这里的一个特点是，这个requireComponent 既是一个方法，也是一个对象，因为它既能像方法一样传递参数进行操作，也可以调用keys这个方法。

* 这里使用的 upperFirst 和 camelCase 是 lodash里面的方法，其中 upperFirst的作用是首字母大写，而camelCase是将字符串转为驼峰命名法，比如空格，-，_将会被划分。

* 在这里，首先fileName是一个文件的路径和名字，所以使用split进行划分/， 然后取出最后一个，然后将后面的文件后缀改为空，然后给camelCase变为驼峰，然后返回的字符串给了upperFirst变为了首字母大写的驼峰。赋值给了componentName。

  * ```
    const componentName = upperFirst(
      camelCase(
        // 获取和目录深度无关的文件名
        fileName
          .split('/')
          .pop()
          .replace(/\.\w+$/, '')
      )
    )
    ```

* 然后就是注册组件了，使用了 Vue.component 方法。进行注册，Vue.component 第一个参数是名字，第二个参数是需要被Vue实例的对象。

  * 这里的唯一的问题就是，对于如果没有使用 export default 导出的方法，貌似不会存在 default 这个对象。但是具体的我们需要在后面才会知道。

  * ```
    Vue.component(
      componentName,
      // 如果这个组件选项是通过 `export default` 导出的，
      // 那么就会优先使用 `.default`，
      // 否则回退到使用模块的根。
      componentConfig.default || componentConfig
    )
    ```



​		**如何实现一个方法带有对象的使用。**

​		这里我的一个想法就是修改原型链了，因为JavaScript任何都是存在原型链的，对于方法来说，也是有一个原型链的。简单来理解，方法既可以使用，又可以当作一个构造器来创建对象。

* 作为一个构造器来说，方法需要记住的是prototype的指向
* 而对于方法的执行来说，需要记住的是 \_\_proto\_\_ 的指向。所以这里对 \_\_proto\_\_ 里面创建了一个方法，然后通过调用这个方法来获取了keys，并且这个本身也是一个方法。

```
function T() {
  return 1;
}

T.__proto__.keys = function () {
  console.log(2);
}
```



# Prop

## Prop的大小写

​		简单来说就是驼峰命名法(camelCase)会在HTML上进行使用时需要转换为短横线分割命名(kebab-case)。因为HTML文档解析是大小写不敏感的，所以 postTitle 会被解析成 posttitle

```
<blog-post :post-title="a"></blog-post>

Vue.component('blog-post', {
  // 在 JavaScript 中是 camelCase 的
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
});
```

​		但是如果是使用的字符串模板，那么就不会有这个限制。

​		意思就是说，使用template这种，模板是字符串的，应该是中间有个步骤在解析的时候会将驼峰自动转换为短横线。所以没有问题。



## Prop类型

* 正常prop在使用的时候是使用的数组，加上字符串进行存储。此时的prop是一个数组的形式。

  * ```
    props: ['title', 'author']
    ```

* 当然我们可以设置prop的值的类型。此时的prop是一个对象的形式。

  * ```
    props: {
      title: String,
      likes: Number,
    }
    ```

  * 对于设置了类型，但是类型对不上的，会报错，但是还是会正常显示，并不会进行类型转换。

* 后面还会将一个写法，props里面的prop也是一个对象，我们后续再进行一个讲解。

  * ```
    props: {
    	propA: {
    		type: String,
    		default: '111',
    		required: true
    	}
    }
    ```



## 传递静态或动态Prop

​		简单来说，prop的传递是通过在使用组件时，添加上了attribute属性在DOM树上，然后再进行的传递，所以这个是可以使用 v-bind 进行绑定的。

​		当然通过这里我也学会一个小case，就是在这样使用的时候，会把42作为一个数字传递过去，而不是字符串，对于布尔值也是一样，如果要传递一个字符串，需要再使用 '' 进行划分。

```
<blog-post :likes="42"></blog-post>
42， false， [12, 41]， {}
```

​		对于要传入一个对象的所有property，我们可以使用 v-bind，进行直接传入。

```
<blog-post v-bind="post"></blog-post>

就等于将 post 里面的对象进行传递

<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>
```

​		当然这里的优先级来说，首先对于子组件没有的prop，是不会进行赋值的，其次对于对象里面存在，同时在外面也进行过一个操作的。比如下面这个情况，是以单独的为重点。

```
<blog-post :post-title="a" v-bind="post"></blog-post>

post: {
  id: 1,
  'post-title': 'xxxx',
}
```

​		其次，就是对于对象里面的属性，在子组件的props里没有，但是确实这个标签的属性的，会被挂载为一个属性。

​		比如：下面这个title就是子组件不存在的，但是属于标签上的属性，我们就可以看到DOM解构上就存在了，同时，我们也发现，id属性应该也是DOM树上的，但是却没有，应该是对于props里面存在的属性会被拦截，只有不在的才会跳出拦截。

```
post: {
  id: 1,
  title: 'xxxx',
}
```



## 单向数据流

​		简单来说，对于这个父子组件的值的传递，所以为了保证子组件意外变更父级组件的状态，我们让子组件不能进行更新，更新会报出警告。并且不能进行更改。

​		然而记住，这个只是对栈进行了一个锁定，并没有对堆进行锁定，所以简单来说就是值类型是会被警告，但是对于引用类型来说，还是可以直接进行修改，并不会爆出警告。

​		这是一个需要记住的问题。



​		对于会进行修改数据，但是又不想污染了父组件的data的，我们这里，有两个方式

* 第一种就是使用了data将props的值重新进行赋值了 **这个子组件接下来希望将其作为一个本地的 prop 数据来使用** 。
  * ```
    props: ['initialCounter'],
    data: function () {
      return {
        counter: this.initialCounter
      }
    }
    ```

* 第二种就是使用计算属性，简单来说就是并没有修改原数据，仅仅只是使用了原数据。常用于 **以一种原始的值传入且需要进行转换** 。



## Prop 验证

​		验证类型是否满足需求，以及是否是必填项。

### 类型检查

* 类型的限制

```
props: {
	A: Number,
	B: [Number, String],
	C: {
		type: String,
	}
}
```

* 是否为必填项和默认值

```
props: {
	A: {
		required: true,
		default: 'AA',
	}
}
```

**注意：**

对象和数组的默认值必须从一个工厂函数获取。简单来说就是需要是一个函数，函数返回一个对象或者数组。

```
default: () => {
	return {
		a: 1
	};
}
```



* 自定义一个验证函数

```
A: {
	validator: function(value) {
		//返回值是 true 和 false
		return true;
	}
}
```



> ​		注意那些 prop 会在一个组件实例创建**之前**进行验证，所以实例的 property (如 `data`、`computed` 等) 在 `default` 或 `validator` 函数中是不可用的。



## 非 Prop 的 Attribute

​		一个非 prop 的 attribute 是指传向一个组件，但是该组件并没有相应 prop 定义的 attribute。

​		这个在前面也有说过，因为prop的传递是在标签上进行的传递，那么怎么区别这个属性是prop的值还是我是要给标签上的值呢。这里就是非Prop的属性

​		简单来理解，就是对于声明的属性，但是却没有prop属性进行接收，那么就会被添加到组件的根元素上。



### 替换/合并已有的 Attribute

​		简单来说就是我在组件的根元素上已经存在了这个属性，但是我在外面使用的使用对这个属性进行了重新的赋值，但是我的需求不是进行覆盖，而是进行合并，此时就是这个 替换/合并 属性

​		但是这里从官网上看出，并没有什么方法，但是对于 class 和 style 这两个属性，我们会进行合并操作，但是对于其他属性，比如 type 等，我们就会出现替换掉的操作。

```
子组件，c-c
div.A[type='AA']

父组件的使用
c-c.B[type='BB']
```



### 禁用 Attribute 继承

​		简单来说就是因为在使用这个组件时，一些没有被prop的属性会被加入根标签，但是有的时候是不想这样的，所以我们可以使用 inheritAttrs: false，代表了继承属性为false。

```
Vue.component('my-component', {
	//这个属性，为false，代表了多余的属性不会被记录在dom树上。（多余是指没有在props定义的），默认为true。
  inheritAttrs: false,
  // ...
})
```

​		当然对于需要被继承的属性，我们可以使用 $attrs 进行获取。这个属性包含了一个属性名和一个属性值。

​		通过 inheritAttrs 和 $attrs 这两个属性，就可以自己决定属性应该被赋予哪个元素。通常用于编写基础组件。

​		下面这个例子，首先，使用了 inheritAttrs 进行了拦截。对于props不存在的属性将不会出现在根元素上。然后再对input标签上添加了一个 v-bind="$attrs" 因为$attrs 属性是一个键值对。然后将其赋值给了input标签上。

```
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
})

<base-input
  label="Username:"
  v-model="username"
  required
  placeholder="Enter your username"
></base-input>
```

```
$attrs 的值。
{
	required: ""
	placeholder: "Enter your username"
}
```

> 注意 `inheritAttrs: false` 选项**不会**影响 `style` 和 `class` 的绑定。

​		通过这个方式来进行操作的话，不用担心哪个是真正的根元素。



# 自定义事件

## 事件名

​		首先要注意一个地方，那就是，对于 vue 的 v-on 监听来说，（语法糖是@）。这个是不会将短横线转为驼峰的。

​		我们从前面可以知道，对于 props 来说，传递的属性如果是短横线，那么在props里面可以使用驼峰来进行处理，但是对于事件来说不行。

​		这里就会好奇为什么prop可以短横线转驼峰，但是事件不能呢。有的说是JavaScript的命名规范里面不能使用短横线命名。

```
this.$emit('myEvent')
<!-- 没有效果 -->
因为 短横线不能转换为驼峰。所以没有效果。
<my-component v-on:my-event="doSomething"></my-component>
```

​		所以对于 @myEvent 将会被转化为 @myevent。

​		所以最好的建议就是全部都是用 kebab-case。这里不但有HTML大小写不敏感的原因，还有对于模板字符串来说。大小写是规定的。



## 自定义组件的v-model 2.2.0+

​		v-model 这个默认是利用名为 value 的prop 和 input 的事件。当然对于input的单选框和其他的会将value属性用于不同的prop。但是我们可以使用 model 选项来解决 v-model的冲突。

​		简单的理解一下下面的内容，prop：checked，这个默认是value，代表了使用props的checked作为v-model的绑定。event：change，默认是input，代表监听的事件名称是change，这些都是可以进行修改的。

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
```

​		比如我这里进行一个修改。就是将 prop 的checked改成了XXX，于是相对应的。props的名字也要有一个XXX的存在。同理。我可以将event改成一个AAA，那么就会将emit提交的事件名称修改为AAA。

```
model: {
  prop: 'XXX',
  event: 'AA'
},
props: {
  XXX: String
},
template: `
  <input
  :value="XXX"
  @change="$emit('AA', $event.target.value)"
  >
`
```

> 注意你仍然需要在组件的 props 选项里声明 checked 这个 prop。



## 将原生事件绑定到组件

​		简单来说就是我想对一个标签进行监听，比如input监听，方法在本身，但是组件的input标签是在子组件的。那么此时我们就可以发现。子组件的输入并不会调用这个方法。

```
<base-input @input="onInput"></base-input>
```

​		我们可以先做一个原生的示例。通过下面这个示例我们可以发现。在输入input的时候会输出1，2 。因为这个是冒泡的原因。所以先进行的子组件的执行，再进行的父组件的执行。

```
<div oninput="console.log(2);">
  <input oninput="console.log(1);"/>
</div>
```

​		那么我们做一个vue组件的示例。并且我们也可以发现。此时就不会执行 myInput 方法了。只会执行组件内部的input的方法。不会执行外部的。原因的话。我们可以知道对于没有props接收的属性一般会被加入根元素，但是我们可以发现对于 监听属性来说，并没有加入到根元素。

```
<div id="app">
  <base-input @input="myInput"></base-input>
</div>

Vue.component('base-input', {
  template: `<input @input="CInput">`,
  methods: {
    CInput() {
      console.log('c');
    }
  }
})
```

​		具体的原因还不清楚。但是要记住就是直接使用监听是不会被监听成功的。有可能是监听要通过JavaScript进行操作，但是组件的元素会被替换，所以监听会失效。

​		因此我们可以使用 .native 修饰符 那么此时就可以将监听放入根元素了。当然，这个的效果简单来说就是会将事件绑定给根元素。但是如果本来根元素不支持这个事件，那么就会静默失败，比如 div的根元素绑定一个focus，就会失败。

```
<base-input @input.native="myInput"></base-input>
```



​		此时，对于根元素并不是我想要绑定的元素，这个时候，我们知道，对于props，有一个叫做。$attrs，和 inheritAttrs 进行搭配，可以使父组件传递的props在子组件能够有一个自己规定的位置出现。那么同时事件也有一个 $listeners 属性，他是一个对象，里面包含了作用在这个组件上的监听器。

```
<div><input v-on="$listeners" @input="CInput"></div>
```

​		但是注意，如果使用了 .native 那么将不会出现在 $listeners 里面。不管 .native 是否会绑定成功。

​		下面这个是使用computed，这里记住一个问题，那就是 下面这个写法是会将 input 的监听进行覆盖的一个操作。但是又通过了 $emit 向上通知了父组件的 input。同理 使用 this.$listeners['input'] 也会覆盖input。

```
computed: {
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
        }
      }
    )
  }
},
```



## .sync修饰符 2.3.0+

​		因为双向绑定会带来维护上的问题。因为对于data的变更不清楚来源。

​		所以我们推荐 update:myPropName 来代替。在一个包含 title 的假设的组件。

```
this.$emit('update:title', newTitle)
```

​		因此就引出了sync修饰符，这算是一个语法糖。

* 对于父组件来说，变化就是将 :title 和 :update:title 进行了合并，简单来说就像是进行了 v-model 的操作一样。

  * ```
    <base-input
      v-bind:title="a"
      v-on:update:title="a"
    ></base-input>
    
    <base-input :title.sync="a"></base-input>
    ```

* 对于子组件来说，没有什么变化， 这里的 :title.sync 就是 @update:title 加上了 :title，所以对于 $emit 也是使用原来的 update:title 进行的提交。

  * ```
    this.$emit('update:title', Math.random());
    ```



在这里，还有一个问题就是。我们可以发现对于 emit提交时带的参数，父组件如果要将方法写在那个标签上进行表达式的执行，那么 $event 就是第一个参数。

```
this.$emit('update:title', 1);

//这里的 $event 就是 子元素在 $emit 提交时的第二个参数，从第二个参数开始的提交就是会传递给父元素的参数。这里只是第二个，如果没有参数的话，就会为undefined
<base-input @update:title="a = $event"></base-input>
```



> 注意带有 `.sync` 修饰符的 `v-bind` **不能**和表达式一起使用 (例如 `v-bind:title.sync=”doc.title + ‘!’”` 是无效的)。取而代之的是，你只能提供你想要绑定的 property 名，类似 `v-model`。



​		当然，这里的 sync修饰符 也可以和 v-bind 进行搭配

```
<text-document v-bind.sync="doc"></text-document>

this.$emit('update:title', 1111);
```



> 将 `v-bind.sync` 用在一个字面量的对象上，例如 `v-bind.sync=”{ title: doc.title }”`，是无法正常工作的，因为在解析一个像这样的复杂表达式的时候，有很多边缘情况需要考虑。



# 插槽

​		简单来说就是，因为Vue的组件化思路，所以很多都是以组件进行的开发。但是为了降低组件和组件的解耦性，就可以使用插槽，将组件之间的联系进一步的降低。

​		在 2.6.0中，提供了新的语法 v-slot 来代替了 slot 和 slot-scope

## 插槽内容

​		

**一个简单的示例**

​		下面的示例就代表了 Your Profile 将会代替 slot这个标签进行显示。当然 插槽的内部可以包含任何模板代码，包括HTML和其他组件，这也是我们常会使用的方法。

```
<navigation-link url="/profile">
  Your Profile
</navigation-link>


<a>
  <slot></slot>
</a>
```



## 编译作用域

​		简单来说，看下面这个代码，这个 a 应该是 父级组件上的 a呢，还是子组件上的a呢。

​		最终的结果就是这个是 父级组件上的a。

```
<div>
	<base-input>{{ a }}</base-input>
</div>
```

这里有条规则

> 父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。



## 后背内容

​		简单来说就是一个插槽的默认值。简单来说就是写在 slot 里面的内容就是 插槽的默认值。

​		如果你在使用组件的时候没有使用插槽，那么就会使用这个默认的内容，并且这个默认的内容的作用域是在子组件的。

​		如果提供有内容，那么便会渲染来取代内容。

```
<div><slot>{{ x }}</slot></div>
```



## 具名插槽

​		简单来说就是使用name将插槽能够指定的使用在某个位置。对于没有使用name的，会有一个默认的name default。 

```
<slot name="header"></slot>
```

​		那么在使用这个具名插槽的时候，我们就可以使用一个 template 进行包裹，然后在里面使用 v-slot指令，并以 v-slot 的参数的形式提供名称。

​		从下面的代码可以看出，对于使用了 template加上v-slot的，就是寻找name相对应的。对于没有使用的就会被作为一个默认，就算中间隔了一个template v-slot 也一样。

```
<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

​		当然，为了语义的方便，我们建议还是可以将默认的放入一个 template v-slot:default，当然，这个default不要也可以，但是如果加了default，那么就不会将其他外层标签也加入默认，但是如果没有加上default的话，那么就会加入默认。

​		比如：

```
<template v-slot:default>
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>
</template>
//下面这个不会显示。
<p>And another one.</p>
```

```
<template>
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>
</template>
//上面的template没有使用default，下面这个会显示出来，因为都被算在了插槽里面
<p>And another one.</p>
```



## 作用域插槽

​		2.6.0 废弃了 slot-scope

​		因为我们可以知道，插槽的默认的作用域是编译时的环境，所以一般常常是父级作用域，那么此时我们要如何将作用域作用域子级。



# 动态组件&异步组件

