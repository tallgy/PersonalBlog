---
layout: draft
title: vue3-getElement伪数组
date: 2021-12-09 13:45:27
tags:
 - bug
categories:
 - bug
---



#  getElementsByClassName 获取的是一个伪数组

## 问题

​		使用的是 document.getElementsByClassName ，但是返回的是一个伪数组，并没有数组的一些方法。最终使用 forof 迭代循环进行的操作。
