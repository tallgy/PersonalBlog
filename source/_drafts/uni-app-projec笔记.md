---
title: uni-app-projec笔记
date: 2022-01-08 16:44:02
tags:
 - 随笔
 - 项目
categories:
 - 随笔
---



# uni-app 项目开发笔记问题

## 不要挂载原型链

将对象挂载在原型链是一个大忌，并且vue2也有警告，因为这样对于维护是一个非常大的问题，使用 forin会将内容显示出来

```
Object.prototype.a = {}
```

