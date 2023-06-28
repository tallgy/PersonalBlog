---
title: router基础
date: 2022-02-07 18:41:53
tags:
 - router
categories:
 - router
---



#  router 的基础学习

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

简述区别：下面的说法很长，并且总结的可能不对：

* $watch 是通过监听的方式进行的监听，所以，只要组件没有被销毁的情况下，监听就有效。所以再从监听组件跳出其他组件时会触发。
* 路由守卫 beforeRouteUpdate 是需要在路由内部的参数变化才会触发。对于跳转去其他路由和从其他路由跳转回来都不会触发。



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



## 路由组件传参

### 通过 props:true 传递给路由组件

```
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const routes = [{ path: '/user/:id', component: User, props: true }]
```

​		那么此时，id 就会通过props传递给路由组件



### 命名视图

​		对于有命名视图的路由，必须给props里面给每个命名视图定义配置

* 如果直接使用 props: true，那么代表里面所有的都为true

* 否则就需要使用对象来对每个命名视图进行处理

```
const routes = [
  {
    path: '/user/:id',
    components: { default: User, sidebar: Sidebar },
    props: { default: true, sidebar: false }
    // 所有的都为true
    // props: true
  }
]
```



### props使用对象

​		当 `props` 是一个对象时，它将原样设置为组件 props。当 props 是静态的时候很有用。

​		就是说，使用的是对象的话，那么对象里面的内容就会作为props传递给路由组件。同时如果此时你的路由组件是携带了参数的。那么将不会传递给路由组件了。

```
const routes = [
  {
    path: '/promotion/from-newsletter',
    component: Promotion,
    props: { newsletterPopup: false }
  }
]
```



### 函数模式

注意：

* 对于函数，参数是route
* 返回值，如果是 Boolean 值，并不会变成 props: true 这两个并不是一个意思。所以不会起作用。
* 返回值，常是一个对象。然后按照对象的形式来处理

```
const routes = [
  {
    path: '/search',
    component: SearchUser,
    props: route => ({ query: route.query.q })
  }
]
```



## 不同的历史模式

### Hash模式

​		hash 模式是用 `createWebHashHistory()` 创建的：

```
import { createRouter, createWebHashHistory } from 'vue-router'

const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    //...
  ],
})
```

​		hash模式就是通过在url后面使用一个hash字符 #。然后因为哈希字符并不会发送到服务器，所以不会出现找不到url的情况。但是对于 SEO 和 看来说，确实有点影响。



### HTML5模式，history模式

​		用 `createWebHistory()` 创建 HTML5 模式，推荐使用这个模式：

```
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    //...
  ],
})
```

​		当使用这种历史模式时，URL 会看起来很 "正常"，例如 `https://example.com/user/id`。漂亮!

​		不过，问题来了。由于我们的应用是一个单页的客户端应用，如果没有适当的服务器配置，用户在浏览器中直接访问 `https://example.com/user/id`，就会得到一个 404 错误。这就丑了。

​		不用担心：要解决这个问题，你需要做的就是在你的服务器上添加一个简单的回退路由。如果 URL 不匹配任何静态资源，它应提供与你的应用程序中的 `index.html` 相同的页面。漂亮依旧!



​		简单来说就是，因为history模式的 url 是不带hash，所以我们知道从一个浏览器里面输入一个正常的url，他会找到对应的服务器并发送请求，那么类似于这种的： https://example.com/usre/id 他就会先进入对应的。



### 服务器配置实例

​		**注意**：以下示例假定你正在从根目录提供服务。如果你部署到子目录，你应该使用[Vue CLI 的 `publicPath` 配置](https://cli.vuejs.org/config/#publicpath)和相关的[路由器的 `base` 属性](https://router.vuejs.org/zh/api/#createwebhistory)。你还需要调整下面的例子，以使用子目录而不是根目录（例如，将`RewriteBase/` 替换为 `RewriteBase/name-of-your-subfolder/`）。



#### Apache

​		没有学过，可以说是完全看不懂了。

​		也可以使用 [`FallbackResource`](https://httpd.apache.org/docs/2.2/mod/mod_dir.html#fallbackresource) 代替 `mod_rewrite`。

```
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```



#### nginx

​		实不相瞒，nginx，我也没有学过。。

```
location / {
  try_files $uri $uri/ /index.html;
}
```



#### 原生 Node.js

​		简单来说，就是先建立监听。

​		然后使用 createServer，这个方法就会将所有的请求都会进行处理了。无论是什么请求，都会进行一个处理。那么我们思考这个算是一个拦截吗，因为既然什么请求都可以进行处理。那么就可以先做一个请求的拦截。当然，这样写符不符合规范不清楚。但是确实做拦截应该是可以的。

​		这里的内容就是，收到请求，然后将index文件里的内容进行读取，然后再通过写响应头，并将内容进行返回。

```
const http = require('http')
const fs = require('fs')
const httpPort = 80

http
  .createServer((req, res) => {
    fs.readFile('index.html', 'utf-8', (err, content) => {
      if (err) {
        console.log('We cannot open "index.html" file.')
      }

      res.writeHead(200, {
        'Content-Type': 'text/html; charset=utf-8',
      })

      res.end(content)
    })
  })
  .listen(httpPort, () => {
    console.log('Server listening on: http://localhost:%s', httpPort)
  })
```



#### Express + Node.js

​		对于 Node.js/Express，可以考虑使用 [connect-history-api-fallback 中间件](https://github.com/bripkens/connect-history-api-fallback)。



后续还有很多其他的，但是因为其他的我连名字都没有听过了，所以就不写了，直接上官网链接。

```
https://router.vuejs.org/zh/guide/essentials/history-mode.html
```



### caveat

​		这有一个注意事项。你的服务器将不再报告 404 错误，因为现在所有未找到的路径都会显示你的 `index.html` 文件。为了解决这个问题，你应该在你的 Vue 应用程序中实现一个万能的路由来显示 404 页面。

```
const router = createRouter({
  history: createWebHistory(),
  routes: [{ path: '/:pathMatch(.*)', component: NotFoundComponent }],
})
```

​		另外，如果你使用的是 Node.js 服务器，你可以通过在服务器端使用路由器来匹配传入的 URL，如果没有匹配到路由，则用 404 来响应，从而实现回退。查看 [Vue 服务器端渲染文档](https://v3.cn.vuejs.org/guide/ssr/introduction.html#what-is-server-side-rendering-ssr)了解更多信息。

​		这里，为什么使用 /:pathMatch(.*) 来匹配404呢，那是因为，这里路由的选择是一个按照分级来排名的。而这个写法的分数只有20分，基本可以说是，只有其他都匹配不上了，才会匹配上这个。

​		所以常用这个来匹配404路由。

