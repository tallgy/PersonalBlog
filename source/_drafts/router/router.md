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

​		同时使用匹配的，对于 /:path 可以匹配到 /a /a/b 这种多层的嵌套。但是前提是，没有其他的path选择。

​		router 使用的路径匹配算法，其灵感来自于 express



## 路由的匹配语法

```
// /:orderId -> 仅匹配数字
{ path: '/:orderId(\\d+)' },
// /:productName -> 匹配其他任何内容
{ path: '/:productName' },
```

​		现在，转到 `/25` 将匹配 `/:orderId`，其他情况将会匹配 `/:productName`。`routes` 数组的顺序并不重要!

> TIP
>
> 确保**转义反斜杠( `\` )**，就像我们对 `\d` (变成`\\d`)所做的那样，在 JavaScript 中实际传递字符串中的反斜杠字符。



```
{ path: '/:pathMatch', component: About }

这样匹配将不能匹配到 / /asd/s
因为这个正则是默认的([^/]+)，^代表了不匹配括号里面的。然后 +，代表了至少需要匹配到一个以上。所以对于上面的，不会被匹配到。
```

```
{ path: '/:p(.*)' } 
同时我们自己也可以进行覆盖。
```

```
{ path: '/:p*' }

可以匹配到 / /as/a /sa/s
同时我们也可以 把 p 进行正则。那么这样的话，可以多次进行匹配。注意这样生成的params是一个数组。而不是一个字符串。
```

```
const routes = [
  // 仅匹配数字
  // 匹配 /1, /1/2, 等
  { path: '/:chapters(\\d+)+' },
  // 匹配 /, /1, /1/2, 等
  { path: '/:chapters(\\d+)*' },
]
```

​		同时也可以使用  **?** 来代表可选参数。当然使用 * 这个也是可选的。但是使用 ？是0或者1个，不会出现重复。



​		**路径的排名**

```
https://paths.esm.dev/?p=AAMeJSyAwR4UbFDAFxAcAGAIJXMAAA..
```

​		这是一个路径的排名工具，作用的话，我认为应该是通过计算分数。路径的匹配会通过分数进行排序来匹配。



## 嵌套路由

​		简单来说就是，router-view 是会渲染 的顶层的路由组件，但是对于组件内部也使用了路由的。我们就可以使用路由嵌套了。

```
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      {
        // 当 /user/:id/profile 匹配成功
        // UserProfile 将被渲染到 User 的 <router-view> 内部
        path: 'profile',
        component: UserProfile,
      },
      {
        // 当 /user/:id/posts 匹配成功
        // UserPosts 将被渲染到 User 的 <router-view> 内部
        path: 'posts',
        component: UserPosts,
      },
    ],
  },
]
```

​		简单来说就是我们会对于匹配成功的子路由，然后会在 User组件里面的 router-view 进行渲染。

```
children: [
    // 当 /user/:id 匹配成功
    // UserHome 将被渲染到 User 的 <router-view> 内部
    { path: '', component: UserHome },

    // ...其他子路由
],
```

​		同时我们可以使用一个空的 path 来代表子路由的home。但是记住，空的path只是代表匹配了 /user/:id 但是对于 /user/:id/asd 虽然没有在子路由匹配成功。但是进入了子路由的，这个不会被匹配到。



## 编程式导航

​		**在 Vue 实例中，你可以通过 `$router` 访问路由实例。因此你可以调用 `this.$router.push`。**

​		当你点击 `<router-link>` 时，内部会调用这个方法，所以点击	 `<router-link :to="...">` 相当于调用 `router.push(...)` 

```
// 字符串路径
router.push('/users/eduardo')

// 带有路径的对象
router.push({ path: '/users/eduardo' })

// 命名的路由，并加上参数，让路由建立 url
router.push({ name: 'user', params: { username: 'eduardo' } })

// 带查询参数，结果是 /register?plan=private
router.push({ path: '/register', query: { plan: 'private' } })

// 带 hash，结果是 /about#team
router.push({ path: '/about', hash: '#team' })
```

​		**但是注意**，如果提供了 `path`，`params` 会被忽略，上述例子中的 `query` 并不属于这种情况。

```
const username = 'eduardo'
// 我们可以手动建立 url，但我们必须自己处理编码
router.push(`/user/${username}`) // -> /user/eduardo
// 同样
router.push({ path: `/user/${username}` }) // -> /user/eduardo
// 如果可能的话，使用 `name` 和 `params` 从自动 URL 编码中获益
router.push({ name: 'user', params: { username } }) // -> /user/eduardo
// `params` 不能与 `path` 一起使用
router.push({ path: '/user', params: { username } }) // -> /user
```

​		由于属性 `to` 与 `router.push` 接受的对象种类相同，所以两者的规则完全相同。

​		 `router.push` 和所有其他导航方法都会返回一个 *Promise*，让我们可以等到导航完成后才知道是成功还是失败。我们将在 [Navigation Handling](https://router.vuejs.org/zh/guide/advanced/navigation-failures.html) 中详细介绍。



### 替换当前位置

​		使用 replace 来进行跳转，唯一不同的是，它在导航时不会向 history 添加新记录，正如它的名字所暗示的那样——它取代了当前的条目。

```
<router-link :to="..." replace>	
router.replace(...)
```

​		也可以直接在传递给 `router.push` 的 `routeLocation` 中增加一个属性 `replace: true` 

```
router.push({ path: '/home', replace: true })
// 相当于
router.replace({ path: '/home' })
```



### 横跨历史

​		该方法采用一个整数作为参数，表示在历史堆栈中前进或后退多少步，类似于 `window.history.go(n)`。

```
// 向前移动一条记录，与 router.forward() 相同
router.go(1) 右箭头

// 返回一条记录，与router.back() 相同
router.go(-1) 左箭头

// 前进 3 条记录
router.go(3)

// 如果没有那么多记录，静默失败
router.go(-100)
router.go(100)
```



### 篡改历史

​		你可能已经注意到，`router.push`、`router.replace` 和 `router.go` 是 [`window.history.pushState`、`window.history.replaceState` 和 `window.history.go`](https://developer.mozilla.org/en-US/docs/Web/API/History) 的翻版，它们确实模仿了 `window.history` 的 API。

​		同时我刚刚进行了测试，发现了 pushState并没有刷新浏览器，但是更新了url，所以这就是router模仿的。对，使用history.go 以及使用 replaceState 都没有进行刷新，仅仅只是修改了url，因为这个是一个同源的方法。对于不同源的处理会出错。



## 命名路由

​		除了 `path` 之外，你还可以为任何路由提供 `name`。这有以下优点：

- 没有硬编码的 URL
- `params` 的自动编码/解码。
- 防止你在 url 中出现打字错误。
- 绕过路径排序（如显示一个）



```
const routes = [
  {
    path: '/user/:username',
    name: 'user',
    component: User
  }
]
```

```
<router-link :to="{ name: 'user', params: { username: 'erina' }}">
  User
</router-link>

路由将导航到路径 `/user/erina`。
```



## 命名视图

​		有时候想同时 (同级) 展示多个视图，而不是嵌套展示，例如创建一个布局，有 `sidebar` (侧导航) 和 `main` (主内容) 两个视图，这个时候命名视图就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 `router-view` 没有设置名字，那么默认为 `default`。

​		简单来说就是，想要存在多个同级的router-view，可以使用命名视图， 默认名字为 default

```
<router-view class="view left-sidebar" name="LeftSidebar"></router-view>
<router-view class="view main-content"></router-view>
<router-view class="view right-sidebar" name="RightSidebar"></router-view>
```

​		一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 `components` 配置 (带上 **s**)：

```
const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    {
      path: '/',
      components: {
        default: Home,
        // LeftSidebar: LeftSidebar 的缩写
        LeftSidebar,
        // 它们与 `<router-view>` 上的 `name` 属性匹配
        RightSidebar,
      },
    },
  ],
})
```

​		当然也是可以不用全部路由都要渲染的。



### 命名嵌套视图

```
{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions
  }, {
    path: 'profile',
    components: {
      default: UserProfile,
      helper: UserProfilePreview
    }
  }]
}
```



## 重定向和别名

```
使用路由
const routes = [{ path: '/home', redirect: '/' }]

使用命名的路由
const routes = [{ path: '/home', redirect: { name: 'homepage' } }]

通过一个方法，返回一个重定向目标
const routes = [
  {
    // /search/screens -> /search?q=screens
    path: '/search/:searchText',
    redirect: to => {
      // 方法接收目标路由作为参数
      // return 重定向的字符串路径/路径对象
      return { path: '/search', query: { q: to.params.searchText } }
    },
  },
  {
    path: '/search',
    // ...
  },
]
```

​		请注意，**[导航守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)并没有应用在跳转路由上，而仅仅应用在其目标上**。在上面的例子中，在 `/home` 路由中添加 `beforeEnter` 守卫不会有任何效果。

​		在写 `redirect` 的时候，可以省略 `component` 配置，因为它从来没有被直接访问过，所以没有组件要渲染。唯一的例外是[嵌套路由](https://router.vuejs.org/zh/guide/essentials/nested-routes.html)：如果一个路由记录有 `children` 和 `redirect` 属性，它也应该有 `component` 属性。



### 相对重定向

​		也可以重定向到相对位置，这里没有理解

```
const routes = [
  {
    path: '/users/:id/posts',
    redirect: to => {
      // 方法接收目标路由作为参数
      // return 重定向的字符串路径/路径对象
    },
  },
]
```



### 别名

​		重定向是指当用户访问 `/home` 时，URL 会被 `/` 替换，然后匹配成 `/`。那么什么是别名呢？

​		 **将 `/` 别名为 `/home`，意味着当用户访问 `/home` 时，URL 仍然是 `/home`，但会被匹配为用户正在访问 `/`。** 

​		别名就是我从url输入的路径会在别名里面判断是否存在，简单来说就是我多次定义一个path不同，但是component相同等其他相同的一个意思，我们不需要这样重复编写，而是通过别名来进行处理。

```
const routes = [{ path: '/', component: Homepage, alias: '/home' }]
```

​		使别名以 `/` 开头，以使嵌套路径中的路径成为绝对路径。你甚至可以将两者结合起来，用一个数组提供多个别名：

```
const routes = [
  {
    path: '/users',
    component: UsersLayout,
    children: [
      // 为这 3 个 URL 呈现 UserList
      // - /users
      // - /users/list
      // - /people
      // /people 是绝对路径， list 是相对路径
      { path: '', component: UserList, alias: ['/people', 'list'] },
    ],
  },
]
```

​		如果你的路由有参数，请确保在任何绝对别名中包含它们：

```
const routes = [
  {
    path: '/users/:id',
    component: UsersByIdLayout,
    children: [
      // 为这 3 个 URL 呈现 UserDetails
      // - /users/24
      // - /users/24/profile
      // - /24
      { path: 'profile', component: UserDetails, alias: ['/:id', ''] },
    ],
  },
]
```



