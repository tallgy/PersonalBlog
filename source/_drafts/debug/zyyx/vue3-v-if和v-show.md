---
layout: draft
title: vue-v-if和v-show
date: 2021-12-09 10:08:41
tags:
 - bug
categories:
 - bug
---



# v-if 和 v-show



## bug情况

​		弹窗显示的图不正确，图是使用的canvas进行的绘制。

## 原因

​		v-show 的显示隐藏不会刷新 canvas，所以导致了最开始的时候没有获取正确的图，后面也没有进行改变。

## 总结

​		在自己写UI组件时，最好对于带有插槽部分的隐藏使用 v-if，而不是v-show。虽然v-show可以有一个更快的体验，但是v-show仅仅只是 display:none 的操作。

​		很容易出现没有更新的情况，特别是使用了canvas绘制的。

