---
layout: draft
title: vue3-nextTick
date: 2021-12-09 13:30:09
tags:
 - bug
categories:
 - bug
---



#  this.$nextTick

## 情况

​		图形在渲染时，没有渲染出正确的位置



## 原因

​		因为图形的渲染是使用的获取窗口的宽度，对于原生的来说，获取窗口的宽度的操作一般是浏览器会先进行渲染操作，但是这里是使用的vue，vue是先将数据放入了虚拟dom，所以此时获取窗口的宽度时并没有先进行一个渲染的操作。所以需要使用 this.$nextTick()



## 总结

​		对于需要通过真实dom的操作的，建议先使用 this.$nextTick() 里面进行获取。
