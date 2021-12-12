---
`title: Vue-使用-可复用性和组合
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

​		自定义选项默认策略是：简单地覆盖已有值。如果想让自定义选项以自定义逻辑合并，可以向 `Vue.config.optionMergeStrategies` 添加一个函数。

```
Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
  // 返回合并后的值
}
```

​		对于多数值为对象的选项，可以使用与 `methods` 相同的合并策略：

​	简单来说，就是 Vue.config.optionMergeStrategies 这个对象里面存有对 Vue实例的对象的不同的值进行不同的处理。比如 new Vue({ methods: {} }) 那么这个混入的合并策略就是存在于 Vue.config.optionMergeStrategies 的 .methods 这个方法里面。

​	所以对于下面这个例子，就是说。因为 methods 方法的合并策略是，针对对象以组件为主。所以将 myOption 的合并策略和 methods 方法一样。

```
var strategies = Vue.config.optionMergeStrategies
strategies.myOption = strategies.methods
```

​	注意： 

* 对于使用自定的对象，我们可以使用 this.$options 来进行获取。

* 自定义选项合并策略，并不是在使用了 mixins 之后才会调用的。而是只要你定义了 Vue.config.optionMergeStrategies ，并且你的Vue实例上，或者 mixins 上存在了，那么就会进行调用的方法。

  * ```
    const mixin = {
    	op: 1,
    }
    
    //如果，mixins里面不存在，但是实例存在，也会调用，但是toVal会为undefined，同理，mixins存在，但是实例不存在的也会调用，只有都不存在才不会调用。
    Vue.config.optionMergeStrategies.op = function(toVal, fromVal) {
    	log.toVal	//1
    	log.fromVal	//3
    	
    	//这个是返回的结果，一般覆盖的方法就是 return fromVal || toVal;
    	return 123;
    }
    
    new Vue({
    	mixins: [mixin],
    	op: 3,
    })
    ```



## 自定义指令

### 简介

​		vue的默认内置指令就是 v-show 和 v-model 等。但是Vue也提供了自定义注册内置指令。

​	简单的示例：在进行注册之后，便可以使用 v-focus property 了。

```
//注册一个全局的自定指令。 v-focus
Vue.directive('focus',{
	当，被绑定的元素被插入到dom中时。
	inserted: function(el) {
		聚焦元素。
		el.focus()
	}
})
```

​	当然，如果要注册局部指令，组件也接受一个 directives 的选项。

```
new Vue({
	directives: {
		focus: {
			inserted(el) {
				el.focus()
			}
		}
	}
})
```



### 钩子函数

​		一个自定义指令对象可以提供如下钩子函数（均是可选）：

* bind：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
* inserted：被绑定元素插入父节点时调用，仅保证父节点存在，但不一定已被插入到文档中。
* update：所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指定的值也可以放生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新。

* componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
* unbind：只调用一次，指令与元素解绑时调用。



### 钩子函数参数

​		指令钩子函数会被传入一下参数：

* el： 指令所绑定的元素，可以用来直接操作 DOM
* binding： 一个对象，包含一下property：
  * name： 指令名，不包括 v- 前缀。
  * value： 指令的绑定值，例如： v-my-directive=“1+1” ，绑定值为 2
  * oldValue： 指定绑定的前一个值，仅在 update 和 component Updated 钩子中可用。无论值是否改变都可用。
  * expression：字符串形式的指令表达式。例如 v-ssss="1+1"，表达式为 1+1
  * arg：传给指令的参数，可选。 v-ss:foo， 参数为 foo
  * modifiers：一个包含修饰符的对象。 v-ssss.foo.bar 中，修饰符对象为 { foo: true, bar: true }
* vnode：Vue 编译生成的虚拟节点。
* oldVnode：上一个虚拟节点。仅在 update 和 componentUpdated 钩子中 可以用。

> 除了 `el` 之外，其它参数都应该是只读的，切勿进行修改。如果需要在钩子之间共享数据，建议通过元素的 [`dataset`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/dataset) 来进行。



样例：

```
<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>
```

```
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})
```

```
name: "demo"
value: "hello!"
expression: "message"
argument: "foo"
modifiers: {"a":true,"b":true}
vnode keys: tag, data, children, text, elm, ns, context, fnContext, fnOptions, fnScopeId, key, componentOptions, componentInstance, parent, raw, isStatic, isRootInsert, isComment, isCloned, isOnce, asyncFactory, asyncMeta, isAsyncPlaceholder
```



#### 动态指令参数

​		指令的参数是动态的。例如：在 v-sss:[argument]="value"中，argument参数可以根据组件实例数据进行更新。

​	v-directive:[arg]="value"

```
<body>
  <div id="sudoku-demo" class="demo">
    <p v-pin:[direction]="200"> i am 200px to the xxx</p>
  </div>

  <script>
  // el 进行直接操作DOM。然后如果使用了binding.arg，那么将使用这个方向，如果没有，则使用 left。
    Vue.directive('pin', {
      bind(el, binding, vnode) {
        el.style.position = 'fixed'
        const direction = binding.arg || 'left'
        el.style[direction] = binding.value + 'px'
      }
    })

    new Vue({
      el: "#sudoku-demo",
      data() {
        return {
          direction: 'top'
        }
      }
    });
  </script>
</body>
```



### 函数简写

​		很多时候，你可能想在 bind 和 update 时出发相同的行为，而不关心其他的钩子。那么就可以使用函数简写

​	下面这个写法，第二个参数将不是一个对象，而是直接作为一个函数作为一个参数，此时这个函数就将在 bind 和 update 里面进行调用，而不会在其他钩子进行调用。

```
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```



### 对象字面量

​		如果一个指令需要多个值，那么可以传入一个 JavaScript 对象字面量。记住，指令函数能够接受所有的合法的 JavaScript 表达式。

```
<div v-demo="{ color: 'white', text: 'hello!' }"></div>

Vue.directive('demo', function (el, binding) {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text)  // => "hello!"
})
```



## 渲染函数 和 JSX

### 基础

​		Vue 推荐绝大多数情况下使用模板来创建你的 HTML。然而在一些场景中，你真的需要 JavaScript 的完全编程能力。

​	比如下面这个使用了渲染函数的方法，可以根据传递的level，来渲染出不同的 h标签，然后将默认插槽传递入子节点数组。

```
Vue.component('heading', {
	render: (createElement) {
		return createElement(
			'h' + this.level,	//标签名称
			this.$slots.default	//子节点数组
		)
	},
	props: {
		level: {
			type: Number,
			required: true
		}
	}
})
```

​	当然，我这里也可以使用 component 标签加上 is 属性来进行代替

```
Vue.component('heading', {
	template: `<component :is="'h' + level"><slot></slot></component>`,
	props: ['level'],
})
```



### 节点、树以及虚拟DOM

​		首先，浏览器在执行html文档时，会建立一个 DOM节点树。来保持追踪所有内容。

​	每一个元素是一个节点，每一段文字也是一个节点，连每一个注释都是一个节点的存在。

​	浏览器在进行节点的追踪时会创建一个节点的树。



#### 虚拟DOM

​		Vue 通过建立一个 虚拟DOM 来追踪自己要如何改变真实的DOM。

​	下面这个方法的返回值不是实际的DOM元素，而是一个节点的描述信息，我们也称之为 虚拟DOM。常常简写为 VNode。

```
return createElement('he', this.blogTitle)
```



### createElement参数

​		接下来需要熟悉 createElement 函数中使用模板中的那些功能。

```
return VNode
crateElement(
	{ String|Object|Function }
	'div',
	
	{ Object }
	一个与模板中 attribute 对应的数据对象。可选。
	{
		// 详情见下一节
	},
	
	{ String|Array }
	子级虚拟节点 由 createElement 构成。
	也可以使用字符串来生成 文本虚拟节点， 可选
	[
		'先写一些文字',
		createElement('h1', '一则头条')
		createElement(myComponent, {
			props: {
				someProp: 'foobar'
			}
		})
	]
)
```



#### 深入数据对象

​		v-bind:class 和 v-bind:style 在模板语法中会被特别对待一样。VNode 数据对象也有顶层的字段。允许绑定普通的HTML属性，也允许绑定 inner HTML 属性（这个会覆盖 v-html 指令）

​	这些属性 可以通过 this._vnode 进行查看。

```
{
	// 和 v-bind:class API 相同
	// 接受一个 String | Objecgt | Array
	'class': {
		foo: true,
		bar: false
	},
	// 和 v-bind:style API 相同
	// 接受一个 String | Object | Array
	style: {
		color: 'red'
	},
	//普通的 HTML attribute
	attrs: {
		id: 'foo'
	}
	props: {
		myProp: 'bar'
	},
	// DOM property
	domProps: {
		innerHTML: 'baz'
	},
	// 事件监听器在 'on' 内
	// 但不再支持 v-on:keyup.enter 这样的修饰器
	需要在函数中手动检查 keyCode
	on: {
		click: this.clickHandler
	},
	// 仅用于组件，用于监听原生事件，而不是组件内部使用
	// vm.$emit 触发的事件
	nativeOn: {
		click: this.nativeClickHandler
	},
	// 自定义指令，无法对 binding 的 oldValue 进行赋值，因为 Vue自动进行了 同步。
	directives: {
		{
			name: 'directive',
			value: '2',
			expression: '1+1',
			arg: 'foo',
			modifiers: {
				bar: true
			}
		}
	}
}
```





#### 完整示例

#### 约束

### 使用 JavaScript 代替模板功能

#### v-if 和 v-for

#### v-model

#### 事件 & 按键修饰符

#### 插槽

### JSX

### 函数式组件

#### 向子元素或子组件传递 attribute 和 事件

#### slots() 和 children 对比

### 模板编译



## 插件

## 过滤器
