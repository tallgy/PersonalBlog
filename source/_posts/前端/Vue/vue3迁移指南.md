---
layout: draft
title: vue3迁移指南
date: 2021-12-14 14:51:47
tags:
 - Vue
 - Vue3
 - 文档
categories:
 - Vue
 - Vue3
---



# 介绍

## 快速开始

​		CDN，Codepen，CodeSandbox

- 通过 CDN：`<script src="https://unpkg.com/vue@next"></script>`
- [Codepen](https://codepen.io/yyx990803/pen/OJNoaZL) 上的浏览器内试验田
- [CodeSandbox](https://v3.vue.new/) 上的浏览器内沙盒

使用脚手架 vite 或者 vue-cli

## 用于迁移的构建版本

​		如果你打算要将一个基于 Vue 2 的项目或者库升级到 Vue 3，我们提供了一个与 Vue 2 API 兼容的 Vue 3 构建版本，详情见[用于迁移的构建版本](https://v3.cn.vuejs.org/guide/migration/migration-build.html)。

### 值得注意的新特性

* 组合式API
  * 使用了 setup 方法。在组件创建之前执行，一旦 props 被解析就将作为组合式API的入口。
  * `setup` 选项是一个接收 `props` 和 `context` 的函数，我们将在[之后](https://v3.cn.vuejs.org/guide/composition-api-setup.html#参数)进行讨论。
  * 我们将 `setup` 返回的所有内容都暴露给组件的其余部分 (计算属性、方法、生命周期钩子等等) 以及组件的模板。

```
// 在我们的组件内
setup (props) {
  let repositories = []
  const getUserRepositories = async () => {
    repositories = await fetchUserRepositories(props.user)
  }

  return {
    repositories,
    getUserRepositories // 返回的函数与方法的行为相同
  }
}
```

​		但是此时的 repositories 变量是非响应式的。此时就需要使用 ref 函数使任何响应式变量在 ref 函数中起作用。

带 ref 的响应式变量

​		ref 接收参数并将其包裹在一个带有 value property 的对象中返回，然后可以使用该 property 访问或更改响应式变量的值：

​	将值封装在对象中，看似没有必要，但是为了保持 JavaScript 中不同数据类型的行为统一，这个是必须的。为了将所有类型都变成引用传递，而不是分为了值传递和引用传递。

```
const counter = ref(0)
```





# 用于迁移的构建版本

​		



# 细节
