---
title: Vue-使用-内在
date: 2021-10-29 23:16:15
tags:
 - Vue
 - Vue2
 - 文档
categories:
 - Vue
 - Vue2
---



#  内在

## 深入响应式原理

​		在前面我们可以知道，当我们修改了data里面的数据时，视图也会进行更新。这样使得状态的管理非常的直接。

### 如何追踪变化

​		Vue会将data里的数据使用 Object.defineProperty ，将这些 property 全部转为 getter/setter。因为 Object.defineProperty 是一个无法shim的特性。所以这也是Vue不支持IE8以及更低的版本的原因。

​		getter/setter 在 property 被访问和修改的时候通知变更。

​		每一个组件的实例都对应一个 watcher 实例，他会在组件渲染的过程中把 接触过的数据 property 记录为依赖。然后当 setter 触发时，通知 watcher，从而重新渲染。

<img src="Vue2-教程-内在\image-20211214103613477.png" alt="image-20211214103613477" style="zoom:67%;" />

​	data会被变更为 getter和setter，然后在视图使用到数据时，便会调用getter，然后添加入了一个dep依赖(这个是使用一个数组进行维护)。当数据发生改变时，会通过notify通知给watcher，然后watcher将会把dep数组的数据重新渲染。



### 检测变化的注意事项

​		由于JavaScript的限制，Vue不能检测数组和对象的变化。

#### 对于对象

​		对于对象来说，无法检测 property 的添加或移除。所以property必须在data对象上存在才能让vue将它转换为响应式的。

```
var vm = new Vue({
  data:{
    a:1
  }
})

// `vm.a` 是响应式的

vm.b = 2
// `vm.b` 是非响应式的
```

​		对于已经创建的实例，Vue不允许动态添加根级别的响应式 property。但是可以使用 Vue.set(object, propertyName, value) 方法向嵌套对象添加响应式 property。

```
Vue.set(vm.someObject, 'b', 2)
```

​	当然可以使用 vm.$set 实例方法，这也是全局 Vue.set 方法的别名

```
this.$set(this.someObject, 'b', 2)

this.$set(this.a, 'b', 123)
```

​		有时你可能需要为已有的对象赋值多个新的property，比如使用 Object.assign() 或者 _.extend()。但是，这样添加并不会触发事件。此时需要将新的对象返回覆盖才能触发。

```
// Object.assign(this.someObject, {a:1, b:2})
this.someObject = Object.assign({}, this.someObject, {a:1, b:2})
```

​	通过这个方法，我们可以知道几个知识点：

* Object.assign 是将第二个参数及其以后的赋值给第一个，对于重复的，会被覆盖，后面的会覆盖前面的。

* 第二点，这里为什么会被更新呢，其实并不是将someObject的内部的对象进行了 getter和setter 操作，而是因为someObject被修改了地址，所以我们也可以知道，这个 Object.defineProperty 是将这个值所指向的位置的监听。

  * 因为对象是堆的存放，而 Object.defineProperty 监听的是栈的值，所以对于对象内部的修改不会被监听到。而将新的对象赋值给了这个对象，此时对象的普通赋值是一个浅拷贝，只拷贝栈值，所以栈被修改了，导致了内部的数据全部都会进行一个监听。

  * 而如果使用 this.$set(this.someObject, 'o', value)的话，仅仅只是将这个o的指向进行了监听，而不会监听其他的错误操作。比如：

    * ```
      this.a.c = 123
      // 此时这个d加入了监听，但是对于c来说，并不会加入监听。因为c是一个错误添加方式。虽然不会进行监听，但是 {{ a }}这样的输出还是会将c进行显示的，因为输出的是a，a.c虽然没有加入监听，但是仍然是属于a的。
      this.$set(this.a, 'd', 1)
      
      但是使用 Object.assign
      这样，对于c来说也会进行监听，因为此时是将原来的this.a进行了栈的修改，对象已经不是一个对象了，此时Vue就会进行一个递归的操作，将新的对象全部进行一个监听。
      this.a = Object.assign({}, this.a, {a:1, b:2})
      ```



#### 对于数组

​		Vue对于数组的监听是将数组的方法进行了一个覆盖，在使用原来的方法的同时，也在里面加入了监听的方法，所以对于直接修改数组下标的将不会进行修改。

​		Vue不能检测以下数组的变动：

* 通过索引直接进行修改 arr[index] = newValue
* 修改数组的长度 arr.length = newLength

对于使用 下标进行修改的

```
这里可以使用，可以考虑JavaScript数组也是对象的想法
this.$set(this.arr, index, newValue)
```

```
this.splice(index, 1, newValue)
```

对于修改数组长度的

```
this.arr.splice(newLength)
```



### 声明响应式 property

​		我们可以发现，前面的添加数据的方式，要不就是数组方法的添加，要不就是在对象里面进行添加，而并没有直接新添加一个对象的形式。由此可以知道：Vue是不运行动态添加根级响应式property的。对于根级响应式必须要提前进行申明。



### 异步更新队列

​		Vue在更新数据是异步进行更新的。对于数据的变化，Vue将开启一个队列。缓冲同一个事件循环的所有数据变更。如果watcher被多次触发，将只会被推入到队列中一次。这样可以不必要的操作和DOM的计算。然后在下一个事件循环中，在刷新队列执行的实际操作。 Promise.then() 、 MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0) 进行代替。

​		当然这样虽然有了好处，节省了性能，但是同时，如果我们要进行dom操作时将会出现问题，因为异步的原因，dom获取的值是在上一次所剩下的。此时便有了一个方法，this.$nextTick(callback)，将方法写入this.$nextTick，然后这个方法将会在下次事件循环，dom操作之后进行获取。

```
methods: {
    updateMessage: function () {
        this.message = '已更新'
        console.log(this.$el.textContent) // => '未更新'
        this.$nextTick(function () {
	        console.log(this.$el.textContent) // => '已更新'
        })
    }
}
```

​	并且 this.$nextTick 返回的是一个 promise 对象，所以你可以使用新的 async/await 语法完成相同的事情

```
methods: {
  updateMessage: async function () {
    this.message = '已更新'
    console.log(this.$el.textContent) // => '未更新'
    await this.$nextTick()
    console.log(this.$el.textContent) // => '已更新'
  }
}
```

