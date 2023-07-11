---
title: webpackloader和plugin
date: 2023-07-11 18:02:28
tags:
 - webpack
categories:
 - webpack
---

# 为什么要分 loader 跟 plugin 而不是合在一起？


```
两者都是为了扩展 webpack 的功能。loader 它只专注于转化文件（transform）这一个领域，完成压缩，打包，语言翻译; 而 plugin 不仅只局限在打包，资源的加载上，还可以打包优化和压缩，重新定义环境变量等
loader 运行在打包文件之前（loader为在模块加载时的预处理文件）； plugins 在整个编译周期都起作用
一个 loader 的职责是单一的，只需要完成一种转换。一个 loader 其实就是一个 Node.js 模块。当需要调用多个 loader 去转换一个文件时，每个 loader 会链式的顺序执行
在 webpack 运行的生命周期中会广播出许多事件， plugin 会监听这些事件，在合适的时机通过 webpack 提供的 API 改变输出结果
```
