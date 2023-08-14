---
title: Vue和React的区别
date: 2023-07-16 16:42:40
tags:
categories:
---

# Vue 和 React 的区别

## 响应式原理不同

Vue 是通过 Object.definedProperty 和 Proxy 对数据进行依赖采集。然后框架内部自动进行渲染更新
React 是通过调用 SetState 方法，框架内部通过方法的调用来进行更新响应式的渲染


## 监听数据变化的实现原理不同

Vue 通过 getter/setter以及一些函数的劫持。
React 默认是通过比较引用的方式（diff）进行的

## 组件写法不同


## Diff算法不同

vue和react的diff算法都是进行同层次的比较，主要有以下两点不同：

vue对比节点，如果节点元素类型相同，但是className不同，认为是不同类型的元素，会进行删除重建，但是react则会认为是同类型的节点，只会修改节点属性。
vue的列表比对采用的是首尾指针法，而react采用的是从左到右依次比对的方式，当一个集合只是把最后一个节点移动到了第一个，react会把前面的节点依次移动，而vue只会把最后一个节点移动到最后一个，从这点上来说vue的对比方式更加高效。

## 核心思想不同


## 数据流不同

Vue 是支持双向绑定的。
React 追求单向数据流，称之为onChange/setState()模式。

## 组合不同功能的方式不同


## 组件通信方法不同

Vue：
* 父组件通过props向子组件传递数据
* 子组件通过事件向父组件发送消息
* 通过 provide/inject 来实现父组件向子组件注入数据，可以跨越多个层级。

React：
* 父组件通过props可以向子组件传递数据或者回调
* 可以通过 context 进行跨层级的通信，这其实和 provide/inject 起到的作用差不多。
* React 本身并不支持自定义事件，而Vue中子组件向父组件传递消息有两种方式：事件和回调函数，但Vue更倾向于使用事件。在React中我们都是使用回调函数。



## 模板渲染方式不同


## 渲染过程不同

Vue可以更快地计算出Virtual DOM的差异，这是由于它在渲染过程中，会跟踪每一个组件的依赖关系，不需要重新渲染整个组件树。
React在应用的状态被改变时，全部子组件都会重新渲染。通过shouldComponentUpdate这个生命周期方法可以进行控制，但Vue将此视为默认的优化。


## 框架本质不同



# 引用

https://worktile.com/kb/ask/19606.html
