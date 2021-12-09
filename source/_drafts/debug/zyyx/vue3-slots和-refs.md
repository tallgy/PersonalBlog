---
layout: draft
title: vue3-$slots和$refs
date: 2021-12-09 11:55:54
tags:
 - bug
categories:
 - bug
---



#  $slots 和 $refs

## 情况

​		对于插槽上的元素，可以使用 $slots 来获取，但是这个获取时，没有获取到真实的dom节点。

​		插槽使用 $slots 没有获取到真实的dom。



## 总结

​		最后选择使用 $refs 来进行替代  $slots 的获取。
