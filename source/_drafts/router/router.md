---
title: router
date: 2022-02-07 18:41:53
tags:
 - router
categories:
 - router
---



#  router 的学习

## router 的基本使用

### HTML

```
<!--使用 router-link 组件进行导航 -->
<!--通过传递 `to` 来指定链接 -->
<!--`<router-link>` 将呈现一个带有正确 `href` 属性的 `<a>` 标签-->
<router-link to="/">Go to Home</router-link>
<router-link to="/about">Go to About</router-link>
```

```
<!-- 路由出口 -->
<!-- 路由匹配到的组件将渲染在这里 -->
<router-view></router-view>
```

### JavaScript

​		先，创建路由

```
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
]

const router = VueRouter.createRouter({
  // 4. 内部提供了 history 模式的实现。为了简单起见，我们在这里使用 hash 模式。
  history: VueRouter.createWebHashHistory(),
  routes, // `routes: routes` 的缩写
})

app.use(router)
```

​		通过JavaScript进行调用

```
this.$router.push('/login')
```

​		请记住，`this.$router` 与直接使用通过 `createRouter` 创建的 `router` 实例完全相同。我们使用 `this.$router` 的原因是，我们不想在每个需要操作路由的组件中都导入路由。



## 动态路由匹配

​		我们可以使用一个动态字段来进行实现，我们称之为 路径参数。

```
const routes = [
  // 动态字段以冒号开始
  { path: '/users/:id', component: User },
]
```

​		现在像 `/users/johnny` 和 `/users/jolyne` 这样的 URL 都会映射到同一个路由。

​		它的 *params* 的值将在每个组件中以 `this.$route.params` 的形式暴露出来。

```
this.$route.params.id
```



### 相应路由参数的变化

​		当用户从 `/users/johnny` 导航到 `/users/jolyne` 时，**相同的组件实例将被重复使用**。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。**不过，这也意味着组件的生命周期钩子不会被调用**。

​		要对同一个组件中参数的变化做出响应的话，你可以简单地 watch `$route` 对象上的任意属性，在这个场景中，就是 `$route.params ` 

