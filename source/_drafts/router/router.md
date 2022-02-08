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

​		**先，创建路由，注意，对于重复的path，前面的会覆盖后面的。意思就是越在上层。优先级越高。** ( 不确定真假。因为使用了正则的出现了不同的结果。后续需要修改这个文字。)

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

通过这个方式添加的参数是直接会放在url路径里面的，所以不会因为刷新而消失



### 相应路由参数的变化

​		当用户从 `/users/johnny` 导航到 `/users/jolyne` 时，**相同的组件实例将被重复使用**。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。**不过，这也意味着组件的生命周期钩子不会被调用**。

​		要对同一个组件中参数的变化做出响应的话，你可以简单地 watch `$route` 对象上的任意属性，在这个场景中，就是 `$route.params ` 

**方法一：我们可以使用 watch 事件，对params进行监听：** 

```
const User = {
  template: '...',
  created() {
    this.$watch(
      () => this.$route.params,
      (toParams, previousParams) => {
        // 对路由变化做出响应...
      }
    )
  },
}
```

​		我们逐行分析，首先，this.$watch 是vue的一个 vm.$watch 的方法。其实就是 watch 属性。

​		然后，我们得知道 this.$watch 的参数，才知道这个作用是什么。

* 参数一： 是一个string类型，是一个表达式。或者是一个函数。通过判断返回值的变化来进行处理。
* 参数二：是一个调用的方法。对应的两个参数是 oldValue 和 newValue。同时会返回一个 unwatch 的方法。调用后将取消监听。

```
var unwatch = vm.$watch('a', cb)
// 之后取消观察
unwatch()
```



**方法二：使用路由守卫 beforeRouterUpdate**：

```
const User = {
  template: '...',
  async beforeRouteUpdate(to, from) {
    // 对路由变化做出响应...
    this.userData = await fetchUser(to.params.id)
  },
}
```



**我们分析两个的区别**：

第一个：是利用路由的复用原因，所以使用watch来监听参数的变化，通过参数的变化，来表示里面的修改。

​		同时使用 watch 进行监听，对于跳转去其他路由，也会进入监听的判断。

​		为什么跳出去的时候会进行一次判断。虽然我们可以知道，监听的是 this.$route.params。这个是一个全局且是一个唯一的对象。

​		就是说。在当前项目里面的params。那么就会是这一个，那么确实，监听有效是应该的。但是我们更应该思考。这个监听我们是在这个里面进行的创建，那么在这个组件被移除之后，监听也会被移除。所以按理，跳转出去之后是不会触发监听的。

​		结果也算是半个很像。因为在跳转出去之后的每一次都不会触发监听。但是，在跳转出去的那一次也会触发。并把目的路由也找到。

​		说明了路由是在destroy之前进行了判断跳转修改了。

第二个：是使用的路由守卫。通过每次跳转路由时，会经过的路由守卫来判断。

​		通过使用路由守卫来进行判断的。跳转去其他路由，并不会触发路由守卫。

​		但是跳转时机 beforeRouteUpdate 会在监听之前进行输出。

​		那么我们就需要思考了，首先对于 before路由守卫 会在监听前进行调用执行。那么按理说。应该会和监听的输出基本一致。

​		但是最终就是，对于跳转出去的路由并不会触发路由守卫



​		同时我们通过断点调试发现。before路由守卫 执行之后，浏览器的url才会进行改变。意思就是说。watch事件的时候，浏览器的url已经发生改变了。



### 捕获路由

​		简单来说，我们可以通过加入正则表达式来进行。

```
const routes = [
  // 将匹配所有内容并将其放在 `$route.params.pathMatch` 下
  { path: '/:pathMatch(.*)*', name: 'NotFound', component: NotFound },
  // 将匹配以 `/user-` 开头的所有内容，并将其放在 `$route.params.afterUser` 下
  { path: '/user-:afterUser(.*)', component: UserGeneric },
]
```

​		这里使用了 正则表达式的语法。然后意思就是 :pathMatch 里面的内容要满足正则表达式，对于不满足的。那么就不会进入这个路由。











