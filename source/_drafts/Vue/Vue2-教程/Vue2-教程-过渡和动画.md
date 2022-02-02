---
title: Vue-使用-过渡和动画
date: 2021-10-29 23:15:25
tags:
 - Vue
 - Vue2
 - 文档
categories:
 - Vue
 - Vue2文档
---



# 过渡和动画

## 进入/离开和列表转换

​		Vue提供了多种方法在DOM中的过渡效果

* 自动为CSS过渡和动画应用类
* 集成第三方CSS动画库，如 Animate.css
* 在过渡钩子期间使用 JavaScript 直接操作 DOM
* 集成第三方JavaScript动画库，例如 Velocity.js



### 过渡单个元素/组件

​		Vue 提供了一个`transition`包装组件，允许您在以下上下文中为任何元素或组件添加进入/离开转换：

* 条件渲染（使用`v-if`）
* 条件显示（使用`v-show`）
* 动态组件
* 组件根节点



​		在这里我直接复制官网的例子，并加以一个简单的说明，首先，对于transition来说，需要的是一个根组件（根元素），对于存在多个根组件的，便会只渲染一个，其次，对于不是非根元素进行的渲染显示不会有效果。

```
<div id="demo">
  <button v-on:click="show = !show">
    Toggle
  </button>
  <transition name="fade">
    <p v-if="show">hello</p>
  </transition>
</div>

new Vue({
  el: '#demo',
  data: {
    show: true
  }
})

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
```

当`transition`插入或删除包裹在组件中的元素时，会发生以下情况：

1. Vue 会自动嗅探目标元素是否应用了 CSS 过渡或动画。如果是这样，CSS 过渡类将在适当的时间添加/删除。
2. 如果转换组件提供了[JavaScript hooks](https://vuejs.org/v2/guide/transitions.html#JavaScript-Hooks)，这些 hooks 将在适当的时间被调用。
3. 如果没有检测到 CSS 过渡/动画，也没有提供 JavaScript 钩子，插入和/或移除的 DOM 操作将在下一帧立即执行（注意：这是一个浏览器动画帧，与 Vue 的概念不同`nextTick`）。



#### 过渡的类名

​		在进入/离开的过程中，会有6个 class 切换

* v-enter
  * 进入的起始状态，在插入元素之前添加，在插入元素后移除一帧
* v-enter-active
  * 进入的活动状态，在整个进入阶段应用。插入元素之前添加，在过渡完成时移除。定义进入过渡的事件、延迟和缓动曲线。
* v-enter-to
  * 2.1.8+
  * 进入的结束状态。插入元素后添加一帧（同时 v-enter 删除），在过渡完成时删除。
* v-leave
  * 定义离开过渡的开始状态。在离开过渡被触发时立刻生效。下一帧被移除
* v-leave-active
  * 定义离开过渡生效时的状态。在离开过渡立刻生效，过渡完成被移除，同 v-enter-active
* v-leave-to
  * 2.1.8 +
  * 定义离开过渡的结束状态。离开过渡被触发后下一帧生效（同时 v-leave 被删除）。过渡完成被移除



![image-20211203152116522](D:\Documents\Blog\source\_drafts\Vue2-教程-过渡和动画\image-20211203152116522.png)

​		在这些过渡的切换中，如果你使用一个没有名字的transition，则v-是这些类名的默认前缀。

```
<transition>
    <p v-show="show">hello</p>
</transition>

.v-enter-active,
```

​		如果你使用了 `<transition name="my-transition">` ，那么 `v-enter` 会替换为 `my-transition-enter` 。

​		v-enter-active 和 v-leave-active 可以控制进入/离开过渡的不同的缓和曲线。



<img src="D:\Documents\Blog\source\_drafts\Vue2-教程-过渡和动画\image-20211203154333481.png" alt="image-20211203154333481"  />

​		通过图示可以看出，当元素进行显示时：

* 先插入 v-enter 和 v-enter-active， 
* 然后再下一帧删除 v-enter，出现了 v-enter-to
* 然后结束了 v-enter-active 和 v-enter-to 消失。

v-leave 是一样的效果。



#### CSS过渡

​		和普通的过渡是没有什么区别的。

```
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  animation: bounce-in .5s reverse;
}

@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
```



#### 自定义过渡的类名

​		我们可以通过使用以下的属性来自定义过渡的类名。并且他们的优先级会高于普通的类名。因此对于使用其他第三方css库十分的有用。

```
enter-class
enter-active-class
enter-to-class (2.1.8+)
leave-class
leave-active-class
leave-to-class (2.1.8+)
```

​		这里name就是往常一样的name，然后在元素里面添加了标签，标签是 enter-xx-class，或者是 leave-xx-class 这种。

```
<transition
name="custom-classes-transition"
enter-active-class="animated tada"
leave-active-class="animated bounceOutRight"
>
    <p v-if="show">hello</p>
</transition>
```



#### 同时使用过渡和动画

​		Vue为了能知道过渡的完成，必须设置相应的事件监听器。他可以是 transitionend 或者是 animationend，如果你只是使用了其中一种，Vue可以自动识别并设置监听。

​		但是如果你同时使用了 transition 和 animation，那么这种情况你就需要使用 type 属性来指定Vue需要监听的类型。

​		如果你不指定的话，我从使用上看来他会默认监听的是时间最长的那个，就是延迟时间和运行时间加起来最大的那个。

```
<transition name="slide-fade" type="transition">
    <p v-if="show"> content </p>
</transition>
```



#### 显性的过渡持续时间

​		2.2.0 新增

​		默认情况下，Vue会自动计算出过渡时间。会等待过渡效果的 transitionend 和 animationend 事件。

​		但是，我们可以显性的设置一个过渡的持续时间。

​		下面这个代码的意思就是，对于使用了 duration 设置了显性的过渡时间了的，在到达过渡的时间之后，便会直接进行过渡的结束。

```
<transition :duration="1000">...</transition>

<transition :duration="{ enter: 500, leave: 800 }">...</transition>
```



#### JavaScript 钩子

​		可以在属性中声明 JavaScript 钩子。

​		这里我简单介绍一下 JavaScript钩子的执行时机，虽然通过名字也能够大致猜的出来意思

* before-enter： 简单来说就是在enter启动之前，就像是前面的 v-enter
* enter： 就是在调用期间， v-enter-to，但是这个 JavaScript钩子 没有说版本要求，而 v-enter-to 是2.1.8以上的，所以这个应该是更像 v-enter-active 。
* after-enter： 就是在就是结束时会进行的调用，应该接近于 animationend 或者 transitionend 这两个回调。
* enter-cancalled： 就是在没有调用after-enter的时候就又进行了一次切换，此时就会调用 enter-cancalled 回调。

```
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```

​		这里需要注意，

* 当只用 JavaScript 过渡的时候，**在 `enter` 和 `leave` 中必须使用 `done` 进行回调**。否则，它们将被同步调用，过渡会立即完成。
  * 这个我没有仔细使用过函数进行动画的过渡，而是进行一些其他的操作，但是这里 done的回调是必须的。因为我们可以看到如果不使用done函数，那么便不会调用 after-enter 来结束，所以这个done函数更像是 生成器函数的 next 一样。
  * 这里我使用了css进行过渡，然后JavaScript进行钩子的调用，发现css的过渡没有出现效果，直接只能使用钩子的调用了。就算是使用自定义过渡类名属性 enter-class 这些也没有效果。
* 推荐对于仅使用 JavaScript 过渡的元素添加 `v-bind:css="false"`，Vue 会跳过 CSS 的检测。这也可以避免过渡过程中 CSS 的影响。
* leaveCancelled 只能用于 v-show 中。

```
// ...
methods: {
  // --------
  // 进入中
  // --------

  beforeEnter: function (el) {
    // ...
  },
  // 当与 CSS 结合使用时
  // 回调函数 done 是可选的
  enter: function (el, done) {
    // ...
    done()
  },
  afterEnter: function (el) {
    // ...
  },
  enterCancelled: function (el) {
    // ...
  },

  // --------
  // 离开时
  // --------

  beforeLeave: function (el) {
    // ...
  },
  // 当与 CSS 结合使用时
  // 回调函数 done 是可选的
  leave: function (el, done) {
    // ...
    done()
  },
  afterLeave: function (el) {
    // ...
  },
  // leaveCancelled 只用于 v-show 中
  leaveCancelled: function (el) {
    // ...
  }
}
```



### 初始渲染的过渡

​		通过使用 appear 属性设置节点初始渲染的过渡

​		简单来说就是会在开始的时候有一个过渡的效果。

```
<transition appear>
```

​	可以自定义 css 类名

```
<transition
  appear
  appear-class="custom-appear-class"
  appear-to-class="custom-appear-to-class" (2.1.8+)
  appear-active-class="custom-appear-active-class"
>
```

​	自定义JavaScript 钩子

```
<transition
  appear
  v-on:before-appear="customBeforeAppearHook"
  v-on:appear="customAppearHook"
  v-on:after-appear="customAfterAppearHook"
  v-on:appear-cancelled="customAppearCancelledHook"
>
```

​		在上面的例子中，无论是使用 appear attribute 还是使用 v-on:appear 钩子都会生成初始渲染的过渡。



### 多个元素的过渡

​		对于原生的标签，可以使用 v-if / v-else 进行过渡。

> 当有**相同标签名**的元素切换时，需要通过 `key` attribute 设置唯一的值来标记以让 Vue 区分它们，否则 Vue 为了效率只会替换相同标签内部的内容。即使在技术上没有必要，**给在 `<transition>` 组件中的多个元素设置 key 是一个更好的实践。**



```
<transition>
  <button v-if="isEditing" key="save">
    Save
  </button>
  <button v-else key="edit">
    Edit
  </button>
</transition>
```

​		在某些场景，也可以通过给同一个元素的key属性设置不同的状态来代替 v-if 和 v-else。

```
<transition>
  <button v-bind:key="isEditing">
    {{ isEditing ? 'Save' : 'Edit' }}
  </button>
</transition>
```



#### 过渡模式

​		简单来说就是元素的切换是同步的，所以会出现占位的问题，并且当使用的是 v-if 的时候，开始会占位，但是在过渡结束之后就会出现位置消失，然后后面的元素会补充上去。

​		所以此时我们就需要进行处理。

​		解决方式一般是使用定位的形式，因为定位是脱离了文档流的，所以就不会出现这个问题了。然后此时我们再使用一个 transition 进行过渡的显示便有了效果。



​		当然同时生效的进入和离开的过渡不能满足所有要求，所以Vue提供了**过渡模式** mode

* `in-out` ：新元素先过渡，完成后旧元素过渡离开。
* `out-in` ：旧元素先过渡，完成后新元素过渡进入。

```
<transition name="fade" mode="out-in">
  <!-- ... the buttons ... -->
</transition>
```



### 多组件的过渡

​		我们可以直接使用动态组件来进行组件之间的过渡

```
<transition name="component-fade" mode="out-in">
  <component v-bind:is="view"></component>
</transition>
```

```
new Vue({
  el: '#transition-components-demo',
  data: {
    view: 'v-a'
  },
  components: {
    'v-a': {
      template: '<div>Component A</div>'
    },
    'v-b': {
      template: '<div>Component B</div>'
    }
  }
})
```



### 列表过渡

​		对于使用v-for渲染时的过渡。可以使用 transition-group 组件。

**特点**：

* transition-group 会以一个真实的元素呈现：默认为 span，也可以通过 tag 属性来更换为其他元素。
* 不能使用过渡模式。mode
* 内部元素总是需要提供一个唯一的 key 属性值
* CSS 过渡的类 将会应用在内部元素中，而不是这个 组件/容器 本身。



​		这里，首先name为list，其次使用了tag，默认不使用tag时为span，使用了tag为p，所以就是p标签进行包裹。然后内部是使用span标签进行的for循环。最后，使用transition-group，key值是必须的。

```
<transition-group name="list" tag="p">
	<span v-for="item in items" v-bind:key="item" class="list-item">
		{{ item }}
	</span>
</transition-group>
```



#### 列表的进入/离开过渡

​		列表的进入/离开过渡是和以前的过渡的方式是一样的。只是过渡变成了每一个小组件了。

​		问题就是过渡的平滑展现，比如新增的元素，周围的元素会瞬间移动到他们新布局的位置，而不是平滑的过渡。

```
.list-enter-active, .list-leave-active {
  transition: all 1s;
}
.list-enter, .list-leave-to
/* .list-leave-active for below version 2.1.8 */ {
  opacity: 0;
  transform: translateY(30px);
}
```



#### 排序的过渡

​		可以通过使用 v-move 类来实现列表的移动。

```
<transition-group name="list" tag="ul">
    <li v-for="(item, index) in items" v-bind:key="item" class="list-item">
		{{ item }}
    </li>
</transition-group>

.list-item {
    transition: all 1s;
}

.list-enter,
.list-leave-to {
    opacity: 0;
    transform: translateY(30px);
}
```

​		如果我对列表进行一次排序。就会发现动画的效果。

​	这个看起来很神奇，内部的实现，Vue 使用了一个叫 [FLIP](https://aerotwist.com/blog/flip-your-animations/) 简单的动画队列。

​		当然，前面的style样式对于增加排序时的效果都还是不错，但是对于删除来说，效果就不一样了。

​	于是我们对于要进行删除的元素进行了一个绝对定位。

​	对于为什么要使用绝对定位，那是因为当使用了绝对定位之后，那便不会在同一图层。然后此时后面的元素便会向前移动，然后再通过 transition: all .5s; 来进行一个动态的变化。

```
.list-leave-active {
    position: absolute;
}
```

> 需要注意的是使用 FLIP 过渡的元素不能设置为 `display: inline` 。作为替代方案，可以设置为 `display: inline-block` 或者放置于 flex 中



**多维网格也可以进行过渡**：

```
<!DOCTYPE html>
<html>
  <head>
    <title>List Move Transitions Sudoku Example</title>
    <script src="https://unpkg.com/vue"></script>
    <link rel="stylesheet" type="text/css" href="./style.css" />
  </head>
  <body>
    <div id="sudoku-demo" class="demo">
      <h1>Lazy Sudoku</h1>
      <p>Keep hitting the shuffle button until you win.</p>

      <button @click="shuffle">
        Shuffle
      </button>
      <transition-group name="cell" tag="div" class="container">
        <div v-for="cell in cells" :key="cell.id" class="cell">
          {{ cell.number }}
        </div>
      </transition-group>
    </div>

    <script>
      new Vue({
        el: "#sudoku-demo",
        data: {
          cells: Array.apply(null, { length: 81 }).map(function(_, index) {
            return {
              id: index,
              number: (index % 9) + 1
            };
          })
        },
        methods: {
          shuffle: function() {
            this.cells = this.randomSort(this.cells);
          },
          randomSort(arr) {
            const res = arr.slice();
            for (let i=0; i<arr.length; i++) {
              const index = this.randomIndex();
              [res[i], res[index]] = [res[index], res[i]];
            }

            return res;
          },
          randomIndex() {
            return Math.floor(Math.random() * this.cells.length);
          }
        }
      });
    </script>
  </body>
  
  <style>
.container {
  display: flex;
  flex-wrap: wrap;
  width: 238px;
  margin-top: 10px;
}
.cell {
  display: flex;
  justify-content: space-around;
  align-items: center;
  width: 25px;
  height: 25px;
  border: 1px solid #aaa;
  margin-right: -1px;
  margin-bottom: -1px;
}
.cell:nth-child(3n) {
  margin-right: 0;
}
.cell:nth-child(27n) {
  margin-bottom: 0;
}
.cell-move {
  transition: transform 1s;
}

  </style>
  
</html>

```



#### 交错的过渡

​		通过data属性和JavaScript的通信，可以实现列表的交错过渡。简单来说就是使用js方法进行了渲染。因为 velocity 不能访问，所以没有进行实验。

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="staggered-list-demo">
  <input v-model="query">
  <transition-group
    name="staggered-fade"
    tag="ul"
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <li
      v-for="(item, index) in computedList"
      v-bind:key="item.msg"
      v-bind:data-index="index"
    >{{ item.msg }}</li>
  </transition-group>
</div>

new Vue({
  el: '#staggered-list-demo',
  data: {
    query: '',
    list: [
      { msg: 'Bruce Lee' },
      { msg: 'Jackie Chan' },
      { msg: 'Chuck Norris' },
      { msg: 'Jet Li' },
      { msg: 'Kung Fury' }
    ]
  },
  computed: {
    computedList: function () {
      var vm = this
      return this.list.filter(function (item) {
        return item.msg.toLowerCase().indexOf(vm.query.toLowerCase()) !== -1
      })
    }
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
      el.style.height = 0
    },
    enter: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 1, height: '1.6em' },
          { complete: done }
        )
      }, delay)
    },
    leave: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 0, height: 0 },
          { complete: done }
        )
      }, delay)
    }
  }
})
```



### 可复用的过渡

​		可以使用Vue组件系统的实现复用。只需要将 transition 或者 transition-group 作为根组件。

```
Vue.component('my-special-transition', {
  template: '\
    <transition\
      name="very-special-transition"\
      mode="out-in"\
      v-on:before-enter="beforeEnter"\
      v-on:after-enter="afterEnter"\
    >\
      <slot></slot>\
    </transition>\
  ',
  methods: {
    beforeEnter: function (el) {
      // ...
    },
    afterEnter: function (el) {
      // ...
    }
  }
})
```

​	当然，可以使用函数式组件来完成这个任务。函数式组件我们后面在进行了解。

```
Vue.component('my-special-transition', {
  functional: true,
  render: function (createElement, context) {
    var data = {
      props: {
        name: 'very-special-transition',
        mode: 'out-in'
      },
      on: {
        beforeEnter: function (el) {
          // ...
        },
        afterEnter: function (el) {
          // ...
        }
      }
    }
    return createElement('transition', data, context.children)
  }
})
```



### 动态过渡

​		在Vue中，过渡也是数据驱动的。所以name也可以使用 v-bind 来进行绑定。

```
<transition v-bind:name="transitionName">
```

​		所有的过渡 attribute 都可以动态绑定。因为事件钩子都是方法。所以根据组件的状态不同，过渡也会有不同的表现。



## 状态过渡

​		数据元素本身的特效，这些数据要么本身就以数值形式存储，要么可以转换为数值。

- 数字和运算
- 颜色的显示
- SVG 节点的位置
- 元素的大小和其他的 property



### 状态动画与侦听器

```
<script src="https://cdn.jsdelivr.net/npm/tween.js@16.3.4"></script>
<script src="https://cdn.jsdelivr.net/npm/color-js@1.0.3"></script>

<div id="example-7">
  <input
    v-model="colorQuery"
    v-on:keyup.enter="updateColor"
    placeholder="Enter a color"
  >
  <button v-on:click="updateColor">Update</button>
  <p>Preview:</p>
  <span
    v-bind:style="{ backgroundColor: tweenedCSSColor }"
    class="example-7-color-preview"
  ></span>
  <p>{{ tweenedCSSColor }}</p>
</div>
```

```
var Color = net.brehaut.Color

new Vue({
  el: '#example-7',
  data: {
    colorQuery: '',
    color: {
      red: 0,
      green: 0,
      blue: 0,
      alpha: 1
    },
    tweenedColor: {}
  },
  created: function () {
    this.tweenedColor = Object.assign({}, this.color)
  },
  watch: {
    color: function () {
      function animate () {
        if (TWEEN.update()) {
          requestAnimationFrame(animate)
        }
      }

      new TWEEN.Tween(this.tweenedColor)
        .to(this.color, 750)
        .start()

      animate()
    }
  },
  computed: {
    tweenedCSSColor: function () {
      return new Color({
        red: this.tweenedColor.red,
        green: this.tweenedColor.green,
        blue: this.tweenedColor.blue,
        alpha: this.tweenedColor.alpha
      }).toCSS()
    }
  },
  methods: {
    updateColor: function () {
      this.color = new Color(this.colorQuery).toRGB()
      this.colorQuery = ''
    }
  }
})
```

```
.example-7-color-preview {
  display: inline-block;
  width: 50px;
  height: 50px;
}
```



### 动态状态过渡

​		就像 Vue 的过渡组件一样，数据背后状态过渡会实时更新，这对于原型设计十分有用。当你修改一些变量，即使是一个简单的 SVG 多边形也可实现很多难以想象的效果。



### 把过渡放到组件里

```
<script src="https://cdn.jsdelivr.net/npm/tween.js@16.3.4"></script>

<div id="example-8">
  <input v-model.number="firstNumber" type="number" step="20"> +
  <input v-model.number="secondNumber" type="number" step="20"> =
  {{ result }}
  <p>
    <animated-integer v-bind:value="firstNumber"></animated-integer> +
    <animated-integer v-bind:value="secondNumber"></animated-integer> =
    <animated-integer v-bind:value="result"></animated-integer>
  </p>
</div>
// 这种复杂的补间动画逻辑可以被复用
// 任何整数都可以执行动画
// 组件化使我们的界面十分清晰
// 可以支持更多更复杂的动态过渡
// 策略。
Vue.component('animated-integer', {
  template: '<span>{{ tweeningValue }}</span>',
  props: {
    value: {
      type: Number,
      required: true
    }
  },
  data: function () {
    return {
      tweeningValue: 0
    }
  },
  watch: {
    value: function (newValue, oldValue) {
      this.tween(oldValue, newValue)
    }
  },
  mounted: function () {
    this.tween(0, this.value)
  },
  methods: {
    tween: function (startValue, endValue) {
      var vm = this
      function animate () {
        if (TWEEN.update()) {
          requestAnimationFrame(animate)
        }
      }

      new TWEEN.Tween({ tweeningValue: startValue })
        .to({ tweeningValue: endValue }, 500)
        .onUpdate(function () {
          vm.tweeningValue = this.tweeningValue.toFixed(0)
        })
        .start()

      animate()
    }
  }
})

// 所有的复杂度都已经从 Vue 的主实例中移除！
new Vue({
  el: '#example-8',
  data: {
    firstNumber: 20,
    secondNumber: 40
  },
  computed: {
    result: function () {
      return this.firstNumber + this.secondNumber
    }
  }
})
```



### 赋予设计以生命


