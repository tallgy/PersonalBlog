---
title: router进阶
date: 2022-02-10 13:51:03
tags:
 - router
categories:
 - router
---



 **router进阶学习**

# 导航守卫

​		正如其名，vue-router 提供的导航守卫主要用来通过跳转或取消的方式守卫导航。这里有很多方式植入路由导航中：全局的，单个路由独享的，或者组件级的。



## 全局前置守卫

​		你可以使用 `router.beforeEach` 注册一个全局前置守卫：

```
const router = createRouter({ ... })

router.beforeEach((to, from) => {
  // ...
  // 返回 false 以取消导航
  return false
})
```

​		当一个导航触发时，全局前置守卫按照创建顺序调用。守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于**等待中**。

​		每个守卫方法接收两个参数：

- **`to`**: 即将要进入的目标 [用一种标准化的方式](https://router.vuejs.org/zh/api/#routelocationnormalized)
- **`from`**: 当前导航正要离开的路由 [用一种标准化的方式](https://router.vuejs.org/zh/api/#routelocationnormalized)

​		可以返回的值如下:

- `false`: 取消当前的导航。如果浏览器的 URL 改变了(可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址。
- 一个[路由地址](https://router.vuejs.org/zh/api/#routelocationraw): 通过一个路由地址跳转到一个不同的地址，就像你调用 [`router.push()`](https://router.vuejs.org/zh/api/#push) 一样，你可以设置诸如 `replace: true` 或 `name: 'home'` 之类的配置。当前的导航被中断，然后进行一个新的导航，就和 `from` 一样。

​		如果遇到了意料之外的情况，可能会抛出一个 `Error`。这会取消导航并且调用 [`router.onError()`](https://router.vuejs.org/zh/api/#onerror) 注册过的回调。

​		如果什么都没有，`undefined` 或返回 `true`，**则导航是有效的**，并调用下一个导航守卫



​		以上所有都同 **`async` 函数** 和 Promise 工作方式一样：

```
router.beforeEach(async (to, from) => {
  // canUserAccess() 返回 `true` 或 `false`
  return await canUserAccess(to)
})
```



### 可选参数 next

​		**确保 `next`** 在任何给定的导航守卫中都被**严格调用一次**。

```
// BAD
router.beforeEach((to, from, next) => {
  if (to.name !== 'Login' && !isAuthenticated) next({ name: 'Login' })
  // 如果用户未能验证身份，则 `next` 会被调用两次
  next()
})
```

```
// GOOD
router.beforeEach((to, from, next) => {
  if (to.name !== 'Login' && !isAuthenticated) next({ name: 'Login' })
  else next()
})
```



## 全局解析守卫

​		你可以用 `router.beforeResolve` 注册一个全局守卫。这和 `router.beforeEach` 类似，因为它在 **每次导航**时都会触发，但是确保在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被正确调用**。这里有一个例子，确保用户可以访问[自定义 meta](https://router.vuejs.org/zh/guide/advanced/meta.html) 属性 `requiresCamera` 的路由：

```
router.beforeResolve(async to => {
  if (to.meta.requiresCamera) {
    try {
      await askForCameraPermission()
    } catch (error) {
      if (error instanceof NotAllowedError) {
        // ... 处理错误，然后取消导航
        return false
      } else {
        // 意料之外的错误，取消导航并把错误传给全局处理器
        throw error
      }
    }
  }
})
```

​		`router.beforeResolve` 是获取数据或执行任何其他操作（如果用户无法进入页面时你希望避免执行的操作）的理想位置。



## 全局后置钩子

​		你也可以注册全局后置钩子，然而和守卫不同的是，这些钩子不会接受 `next` 函数也不会改变导航本身：

```
router.afterEach((to, from) => {
  sendToAnalytics(to.fullPath)
})
```

​		它们对于分析、更改页面标题、声明页面等辅助功能以及许多其他事情都很有用。

​		它们也反映了 [navigation failures](https://router.vuejs.org/zh/guide/advanced/navigation-failures.html) 作为第三个参数：

```
router.afterEach((to, from, failure) => {
  if (!failure) sendToAnalytics(to.fullPath)
})
```

​		**navigation failures 的属性** 

​		所有导航失败都会公开`to`和`from`属性以反映当前位置以及失败导航的目标位置：

```
// trying to access the admin page
router.push('/admin').then(failure => {
  if (isNavigationFailure(failure, NavigationFailureType.redirected)) {
    failure.to.path // '/admin'
    failure.from.path // '/'
  }
})
```



## 路由独享的守卫

​		你可以直接在路由配置上定义 `beforeEnter` 守卫：

```
const routes = [
  {
    path: '/users/:id',
    component: UserDetails,
    beforeEnter: (to, from) => {
      // reject the navigation
      return false
    },
  },
]
```

​		`beforeEnter` 守卫 **只在进入路由时触发**，不会在 `params`、`query` 或 `hash` 改变时触发。例如，从 `/users/2` 进入到 `/users/3` 或者从 `/users/2#info` 进入到 `/users/2#projects`。它们只有在 **从一个不同的** 路由导航时，才会被触发。

​		注意：

* beforeEnter 是写在routes里面，而不是组件里面。
* 其次触发时机就是进入路由的时候，但是如果没有离开路由就不会再次触发了。其中对于 参数的变化也是在路由之中的。
* return 值的判定就是导航守卫的判定方法。

​		你也可以将一个函数数组传递给 `beforeEnter`，这在为不同的路由重用守卫时很有用：

```
function removeQueryParams(to) {
  if (Object.keys(to.query).length)
    return { path: to.path, query: {}, hash: to.hash }
}

function removeHash(to) {
  if (to.hash) return { path: to.path, query: to.query, hash: '' }
}

const routes = [
  {
    path: '/users/:id',
    component: UserDetails,
    beforeEnter: [removeQueryParams, removeHash],
  },
  {
    path: '/about',
    component: UserDetails,
    beforeEnter: [removeQueryParams],
  },
]
```

​		请注意，你也可以通过使用[路径 meta 字段](https://router.vuejs.org/zh/guide/advanced/meta.html)和[全局导航守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#global-before-guards)来实现类似的行为。



## 组件内的守卫

​		最后，你可以在路由组件内直接定义路由导航守卫(传递给路由配置的)

### 可用的配置 API

你可以为路由组件添加以下配置：

- `beforeRouteEnter`
- `beforeRouteUpdate`
- `beforeRouteLeave`

```
const UserDetails = {
  template: `...`,
  beforeRouteEnter(to, from) {
    // 在渲染该组件的对应路由被验证前调用
    // 不能获取组件实例 `this` ！
    // 因为当守卫执行时，组件实例还没被创建！
  },
  beforeRouteUpdate(to, from) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 `/users/:id`，在 `/users/1` 和 `/users/2` 之间跳转的时候，
    // 由于会渲染同样的 `UserDetails` 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 因为在这种情况发生的时候，组件已经挂载好了，导航守卫可以访问组件实例 `this`
  },
  beforeRouteLeave(to, from) {
    // 在导航离开渲染该组件的对应路由时调用
    // 与 `beforeRouteUpdate` 一样，它可以访问组件实例 `this`
  },
}
```

​		`beforeRouteEnter` 守卫 **不能** 访问 `this`，因为守卫在导航确认前被调用，因此即将登场的新组件还没被创建。

​		不过，你可以通过传一个回调给 `next` 来访问组件实例。在导航被确认的时候执行回调，并且把组件实例作为回调方法的参数：

```
beforeRouteEnter (to, from, next) {
  next(vm => {
    // 通过 `vm` 访问组件实例
  })
}
```

​		注意 `beforeRouteEnter` 是支持给 `next` 传递回调的唯一守卫。对于 `beforeRouteUpdate` 和 `beforeRouteLeave` 来说，`this` 已经可用了，所以*不支持* 传递回调，因为没有必要了：

```
beforeRouteUpdate (to, from) {
  // just use `this`
  this.name = to.params.name
}
```

​		这个 **离开守卫** 通常用来预防用户在还未保存修改前突然离开。该导航可以通过返回 `false` 来取消。

```
beforeRouteLeave (to, from) {
  const answer = window.confirm('Do you really want to leave? you have unsaved changes!')
  if (!answer) return false
}
```



### 使用组合式 API

​		如果你正在使用[组合 API 和 `setup` 函数](https://v3.vuejs.org/guide/composition-api-setup.html#setup)来编写组件，你可以通过 `onBeforeRouteUpdate` 和 `onBeforeRouteLeave` 分别添加 update 和 leave 守卫。 请参考[组合 API 部分](https://router.vuejs.org/zh/guide/advanced/composition-api.html#导航守卫)以获得更多细节。



## 完整的导航解析流程

* 导航被触发
* 在失活的组件里调用 `beforeRouteLeave` 守卫。
* 调用全局的 `beforeEach` 守卫。
* 在重用的组件里调用 `beforeRouteUpdate` 守卫(2.2+)。
* 在路由配置里调用 `beforeEnter`。
* 解析异步路由组件。
* 在被激活的组件里调用 `beforeRouteEnter`。
* 调用全局的 `beforeResolve` 守卫(2.5+)。
* 导航被确认。
* 调用全局的 `afterEach` 钩子。
* 触发 DOM 更新。
* 调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数，创建好的组件实例会作为回调函数的参数传入。

```
	触发导航，失活调用组件的routeleave，调用全局的beforeEach，重用组件调用routeUpdate，调用路由配置beforeEnter，调用routeEnter，调用全局beforeResolve，调用全局afterEach，调用routeEnter的next回调函数。
```



# 路由元信息

​		有时，你可能希望将任意信息附加到路由上，如过渡名称、谁可以访问路由等。这些事情可以通过接收属性对象的`meta`属性来实现，并且它可以在路由地址和导航守卫上都被访问到。定义路由的时候你可以这样配置 `meta` 字段：

```
const routes = [
  {
    path: '/posts',
    component: PostsLayout,
    children: [
      {
        path: 'new',
        component: PostsNew,
        // 只有经过身份验证的用户才能创建帖子
        meta: { requiresAuth: true }
      },
      {
        path: ':id',
        component: PostsDetail
        // 任何人都可以阅读文章
        meta: { requiresAuth: false }
      }
    ]
  }
]
```

​		首先，我们称呼 `routes` 配置中的每个路由对象为 **路由记录**。路由记录可以是嵌套的，因此，当一个路由匹配成功后，它可能匹配多个路由记录。

​		例如，根据上面的路由配置，`/posts/new` 这个 URL 将会匹配父路由记录 (`path: '/posts'`) 以及子路由记录 (`path: 'new'`)。

​		一个路由匹配到的所有路由记录会暴露为 `$route` 对象(还有在导航守卫中的路由对象)的`$route.matched` 数组。我们需要遍历这个数组来检查路由记录中的 `meta` 字段，但是 Vue Router 还为你提供了一个 `$route.meta` 方法，它是一个非递归合并**所有 `meta`** 字段的（从父字段到子字段）的方法。这意味着你可以简单地写

```
router.beforeEach((to, from) => {
  // 而不是去检查每条路由记录
  // to.matched.some(record => record.meta.requiresAuth)
  if (to.meta.requiresAuth && !auth.isLoggedIn()) {
    // 此路由需要授权，请检查是否已登录
    // 如果没有，则重定向到登录页面
    return {
      path: '/login',
      // 保存我们所在的位置，以便以后再来
      query: { redirect: to.fullPath },
    }
  }
})
```

​		matched 里面的值的path是一个路由定义的样子(/about/:id) 而不是实际的url路由值。



## TypeScript

​		可以通过扩展 `RouteMeta` 接口来输入 meta 字段：

```
// typings.d.ts or router.ts
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    // 是可选的
    isAdmin?: boolean
    // 每个路由都必须声明
    requiresAuth: boolean
  }
}
```



# 数据获取

​		有时候，进入某个路由后，需要从服务器获取数据。例如，在渲染用户信息时，你需要从服务器获取用户的数据。我们可以通过两种方式来实现：

- **导航完成之后获取**：先完成导航，然后在接下来的组件生命周期钩子中获取数据。在数据获取期间显示“加载中”之类的指示。
- **导航完成之前获取**：导航完成前，在路由进入的守卫中获取数据，在数据获取成功后执行导航。

​		从技术角度讲，两种方式都不错 —— 就看你想要的用户体验是哪种。

## 导航完成后获取数据

​		当你使用这种方式时，我们会马上导航和渲染组件，然后在组件的 created 钩子中获取数据。这让我们有机会在数据获取期间展示一个 loading 状态，还可以在不同视图间展示不同的 loading 状态。

```
<template>
  <div class="post">
    <div v-if="loading" class="loading">Loading...</div>

    <div v-if="error" class="error">{{ error }}</div>

    <div v-if="post" class="content">
      <h2>{{ post.title }}</h2>
      <p>{{ post.body }}</p>
    </div>
  </div>
</template>
```

```
export default {
  data() {
    return {
      loading: false,
      post: null,
      error: null,
    }
  },
  created() {
    // watch 路由的参数，以便再次获取数据
    this.$watch(
      () => this.$route.params,
      () => {
        this.fetchData()
      },
      // 组件创建完后获取数据，
      // 此时 data 已经被 observed 了
      { immediate: true }
    )
  },
  methods: {
    fetchData() {
      this.error = this.post = null
      this.loading = true
      // replace `getPost` with your data fetching util / API wrapper
      // 用你的数据获取 util 或 API 替换 `getPost`
      getPost(this.$route.params.id, (err, post) => {
        this.loading = false
        if (err) {
          this.error = err.toString()
        } else {
          this.post = post
        }
      })
    },
  },
}
```

​		简单来说就是创建了 loading error post 三个状态，然后再created的时候创建监听（理由是创建的监听参数的变化，可以重复触发。否则在参数变化时使用的复用将不会触发改变。）使用 immediate 为true，立刻触发。再调用fetch方法。然后再进行调用时设置post，error为null，作用是避免复用的影响，设置loading为true，然后便是正常的获取数据并显示了。



## 导航完成前获取数据

​		通过这种方式，我们在导航转入新的路由前获取数据。我们可以在接下来的组件的 `beforeRouteEnter` 守卫中获取数据，当数据获取成功后只调用 `next` 方法：

```
export default {
  data() {
    return {
      post: null,
      error: null,
    }
  },
  beforeRouteEnter(to, from, next) {
    getPost(to.params.id, (err, post) => {
      next(vm => vm.setData(err, post))
    })
  },
  // 路由改变前，组件就已经渲染完了
  // 逻辑稍稍不同
  async beforeRouteUpdate(to, from) {
    this.post = null
    try {
      this.post = await getPost(to.params.id)
    } catch (error) {
      this.error = error.toString()
    }
  },
}
```

​		这个就是在 beforeRouteEnter里面调用了异步请求，然后再获取到数据之后将数据使用next回调进行调用。同时再beforeRouteUpdate进行更新时使用了es6的async 和 await方法。同时设置this.post为null，然后在调用getPost方法获取数据返回给post。

​		在为后面的视图获取数据时，用户会停留在当前的界面，因此建议在数据获取期间，显示一些进度条或者别的指示。如果数据获取失败，同样有必要展示一些全局的错误提醒。



# 组合式API

## 在 setup 中访问路由和当前路由

​		因为我们在 `setup` 里面没有访问 `this`，所以我们不能再直接访问 `this.$router` 或 `this.$route`。作为替代，我们使用 `useRouter` 函数：

```
import { useRouter, useRoute } from 'vue-router'

export default {
  setup() {
  	// useRouter 和 useRoute 是一个方法。返回一个route 和 router 的一个对象
  	// route 是当前路由的信息， router 主要是原型的方法。和 全局路由信息。
    const router = useRouter()
    const route = useRoute()

	// 
    function pushWithQuery(query) {
      router.push({
        name: 'search',
        query: {
          ...route.query,
        },
      })
    }
  },
}
```

​		`route` 对象是一个响应式对象，所以它的任何属性都可以被监听，但你应该**避免监听整个 `route`** 对象：

```
import { useRoute } from 'vue-router'

export default {
  setup() {
    const route = useRoute()
    const userData = ref()

    // 当参数更改时获取用户信息
    watch(
      () => route.params,
      async newParams => {
        userData.value = await fetchUser(newParams.id)
      }
    )
  },
}
```

​		请注意，在模板中我们仍然可以访问 `$router` 和 `$route`，所以不需要在 `setup` 中返回 `router` 或 `route`。



## 导航守卫

​		











