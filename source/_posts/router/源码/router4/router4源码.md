---
title: router4源码
date: 2022-02-11 19:49:25
tags:
 - router
categories:
 - router
---



#  router4 源码的学习

​		在学习router的时候，发现了一个地方，就是**beforeRouteEnter** 里面添加了next参数，然后就发现了，如果不调用就会出现问题，但是如果不添加next参数就不会出这个问题，然后就准备看看源码，刚开始看的源码时vue2的，所以现在重看一个vue3的。



​		**rotuer.ts / 852** 

```
// check in-component beforeRouteEnter
guards = extractComponentsGuards(
  enteringRecords,
  'beforeRouteEnter',
  to,
  from
)
guards.push(canceledNavigationCheck)
```

* 大概就是调用方法。这里的 **enteringRecords** 参数是通过 769 line 返回值

  * ```
    const [leavingRecords, updatingRecords, enteringRecords] =
          extractChangingRecords(to, from)
    ```

  *  **extractChangingRecords** 方法。1238 line。

    * 记住是返回数组，然后数组是代表了会触发 leaving，enter，update 的记录。代码后面有分析，这里就不占空间了。

  *  **navigate** 方法， 763 line

    * ```
      大概就是，先获取到路由的记录
      
      const [leavingRecords, updatingRecords, enteringRecords] =
            extractChangingRecords(to, from)
      ```

    * ```
      调用 beforeRouteLeave，注意，这里的leavingRecoreds 记录使用了 reverse 进行了逆序。所以触发会从下到上。同时 reverse会修改原数组。
      
      guards = extractComponentsGuards(
        leavingRecords.reverse(),
        'beforeRouteLeave',
        to,
        from
      )
      ```

    * ```
      循环 leaving 记录。
      
      
      // leavingRecords is already reversed
      for (const record of leavingRecords) {
        record.leaveGuards.forEach(guard => {
          guards.push(guardToPromiseFn(guard, to, from))
        })
      }
      ```





## router/ts	createRouter方法	355

### navigate 方法 763

```
// 返回要触发对应声明周期的 路由的记录
const [leavingRecords, updatingRecords, enteringRecords] =
	extractChangingRecords(to, from)
```

```
// all components here have been resolved once because we are leaving


guards = extractComponentsGuards(
  leavingRecords.reverse(),
  'beforeRouteLeave',
  to,
  from
)
```



## **extractChangingRecords** 方法。

* ```
  const enteringRecords: RouteRecordNormalized[] = [] 代表定义的是一个数组。类型是 RouteRecordNormalized 
  ```

* ```
  function extractChangingRecords(
    to: RouteLocationNormalized,
    from: RouteLocationNormalizedLoaded
  ) {
    // 数组类型
    const leavingRecords: RouteRecordNormalized[] = []
    const updatingRecords: RouteRecordNormalized[] = []
    const enteringRecords: RouteRecordNormalized[] = []
  
    // matched 代表的是匹配上的路由部分。
    //这里就是寻找len要比较长的一个。
    const len = Math.max(from.matched.length, to.matched.length)
    for (let i = 0; i < len; i++) {
      // 这里就是 recordFrom 就让他等于 from 的匹配路由
      const recordFrom = from.matched[i]
      //如果路由存在，那么就会
      // 从 to 里面匹配上的路由 进行find方法。轮询查看是否有满足的。
      // isSameRouteRecord 方法就是 判断两个路由是不是一个。这里使用了 aliasOf 定义此记录是否是另一个记录的别名。如果该记录是原始记录，则此属性为 undefined。
      if (recordFrom) {
        // 如果轮询找到了，那么就会把这个 from 的匹配路由push为 update的记录。
        // 如果找不到，说明没有，那么就会 push 一个 leaving记录。
        if (to.matched.find(record => isSameRouteRecord(record, recordFrom)))
          updatingRecords.push(recordFrom)
        else leavingRecords.push(recordFrom)
      }
      // 记录此时的 to 的匹配路由
      const recordTo = to.matched[i]
      // 如果存在
      if (recordTo) {
        // the type doesn't matter because we are comparing per reference
        // 判断原路由是否存在，不存在则会push为 enter 记录
        if (!from.matched.find(record => isSameRouteRecord(record, recordTo))) {
          enteringRecords.push(recordTo)
        }
      }
    }
  
    // 最后返回所有的记录
    return [leavingRecords, updatingRecords, enteringRecords]
  }
  ```

  

## navigationGuards/ts 

### extractComponentsGuards方法 230



### isRouteComponent 方法 357

```
/**
 * Allows differentiating lazy components from functional components and vue-class-component
 * 允许区分惰性组件与功能组件和vue类组件
 *
 * @param component
 */
function isRouteComponent(
  component: RawRouteComponent
): component is RouteComponent {
  // 组件要为对象，且存在 displayName，props，__vccOpts属性
  return (
    typeof component === 'object' ||
    'displayName' in component ||
    'props' in component ||
    '__vccOpts' in component
  )
}
```



### guardToPromiseFn方法 110

```

```

