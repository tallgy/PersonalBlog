---
title: single-spa
date: 2023-08-12 23:08:08
tags:
 - 微前端
categories:
 - 微前端
---

# single-spa

微前端

**微前端作用**

* 解决巨石应用问题
* 多人开发协同问题
* 可以做到一次不会请求整个所有项目的资源，类似于懒加载效果
* 让旧的应用可以附属为新的子应用，方便一些登录态的存储


## 逻辑

single-spa 做为基建

通过 registerApplication 创建子应用
然后通过调用 start 方法，会监听 路由变化
所以此时是有子应用的信息。
  比如在 loadApp 里面的 子应用 url，
  所以其实也可以直接通过 url 访问子应用的信息
start 里面会创建 history 的监听

然后子应用会在 main.js 文件抛出 single-spa 的生命周期方法
处理对应的子应用问题。

### 监听

在这里创建的全局监听 popState、hashchange 事件
对于 pushState 和 replaceState 事件使用的是 window.dispatchEvent 事件添加的。
注意；调用 history.pushState() 或者 history.replaceState() 不会触发 popstate 事件。
single-spa\src\navigation\navigation-events.js


## registerApplication

注册子应用方法

调用方法，放入 app 数组。 app 是一个子应用列表


**参数**

* name 应用名称或者应用配置对象，唯一值
* loadApp 应用的加载方法，是一个 promise
* activeWhen 判断应用是否激活的一个方法，方法返回 true or false
* customProps 传递给子应用的 props 对象


