---
layout: draft
title: vue-css外部引入
date: 2022-01-20 16:48:04
tags:
 - vue
 - 随笔
---



#  vue的css样式 进行外部引入问题

最近在项目开发中出现了如下的问题

​	就是 vue的样式 style 使用 scoped 进行了 私有化属性，但是却发现，我内部的一个组件的样式还是被其他地方影响了，所以记录一下问题原因



原因：

​	因为 style 样式是通过 @import 进行的引入，所以就会出现私有化属性不起作用，原因的话，应该是@imort引入是一个属于重新请求的过程？（应该是）所以就绕过了 vue-loader 进行属性私有化过程，导致了样式变成了全局化



方法：

​	其实也没有什么方法，要不就是直接写在 style 里面，要不就是使用 src进行引入，这样引入的样式就不会出现全局污染

```
<style scoped src="~/static/css/xxx.css"></style>
```



同时也记录一下， style是可以同时写多个的，但是 scripte 一个vue文件里面只能写一个，因为需要 export导出的属性。
