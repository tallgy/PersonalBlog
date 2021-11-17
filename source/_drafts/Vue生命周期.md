---
title: Vue生命周期
date: 2021-11-16 22:15:21
tags:
 - Vue
 - 生命周期
categories:
 - Vue
 - 随笔
---



# Vue生命周期

先来一个直译的形式

```
  创建一个Vue实例 new Vue
  初始化事件和生命周期
  调用 beforeCreate钩子
  初始化注入和反应
  调用 created 钩子
  编译模板阶段
		Has el option 这里是判断是否使用了 el这个属性。如果没有使用那么就是属于未挂载的状态，只有在之后 $mount 被调用时才会进行下一步的编译。
		Has template option，判断是否使用了template参数。如果没有使用template参数，那么就将用el元素时的内部标签，如果使用了template元素，那么就会将template的字符串作为标签覆盖el元素内部的，除非有插槽。
		这里的 outerHTML，我们也知道就是：除了包含innerHTML的全部内容外, 还包含对象标签本身。
	然后就是调用 beforeMount 钩子
	然后就是创建 虚拟DOM 并且替换 el，这里就是实现了将页面的显示。此时这里在显示的时候就已经创建好了虚拟DOM并且还进行了数据和视图的绑定。
	调用 mounted 钩子，但是记住，这里这个钩子并不会保正所有的子组件都被挂载完成，虽然大部分情况都是的。当然可以使用 nexttick 这个函数来保证了所有的组件被渲染。
	现在已经装载完成，处于组件在视图中的情况。此时当数据发生改变会引起 beforeupdate钩子
	beforeUpdate钩子函数，在数据发生改变的时候，进行虚拟dom的渲染和patch之前。
	updated钩子函数，在渲染和patch之后。但是不会保证所有的子组件都刷新。
	当调用了 destroy方法，触发了 beforeDestroy钩子
	拆卸 watchers，子组件和事件监听
	已经被销毁了，调用destroyed
```

<img src="Vue生命周期/lifecycle.png" alt="lifecycle" style="zoom:67%;" />



<img src="Vue生命周期/lifecycle9.jpg" alt="lifecycle9" style="zoom:67%;" />



# 分析每一阶段

