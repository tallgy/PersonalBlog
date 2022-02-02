---
title: Vue-使用-工具
date: 2021-10-29 23:15:41
tags:
 - Vue
 - Vue2
 - 文档
categories:
 - Vue
 - Vue2
---



#  Vue2教程-工具

## 单文件组件

### 介绍

​		在开始的Vue项目中，我们使用 Vue.component 来定义全局组件，然后使用 new Vue({el: '#xx'}) 在每个页面内指定一个容器元素。

* **全局定义**：强制要求每个 component 中的命名不重复
* **字符串模板**：缺乏语法高亮，在HTML有多行的时候，需要用到 \
* **不支持CSS**：意味着当 HTML 和 JavaScript 组件化时，CSS 明显被遗漏
* **没有构建步骤**：限制只能使用 HTML 和 ES5 JavaScript，而不能使用预处理器，如 Pug(formerly Jade) 和 Babel

​	因此对应的 .vue 单文件组件便解决了上述的方法。并且还可以使用 webpack 和 Browserify 等构建工具。

#### 怎么看待关注点分离

​		关注点分离不等于文件类型分离。现代的UI开发中，相比于将代码分成三大层次相互交织，我们将其划分为松散耦合的组件在将其组合起来更合理一些。

​		在一个组件中，模板，逻辑和样式都是内部耦合的。但是组件和组件之间时解耦的。

​	当然，我们也仍然可以将 JavaScript、css 分离成独立的文件然后做到热重载和预编译。



### 起步

#### 例子沙箱

#### 针对刚接触JavaScript模块开发系统的用户

#### 针对高级用户

​		CLI 会为你搞定大多数工具的配置问题，同时也支持细粒度自定义[配置项](https://cli.vuejs.org/zh/config/)。

​		有时你会想从零搭建你自己的构建工具，这时你需要通过 [Vue Loader](https://vue-loader.vuejs.org/zh/) 手动配置 webpack。关于学习更多 webpack 的内容，请查阅[其官方文档](https://webpack.js.org/configuration/)和 [Webpack Academy](https://webpack.academy/p/the-core-concepts)。

​	分别看看上面的 cli loader webpack 来进行详细的了解。



## 测试

## TypeScript 支持

## 生成环境部署

### 开启生产环境模式

### 模板预编译

### 提取组件的CSS

### 跟踪运行时错误
