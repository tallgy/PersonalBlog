---
title: JavaScript-Promise.all源码
date: 2022-04-01 14:28:27
tags:
 - JavaScript
 - Promise
categories:
 - JavaScript
 - Promise
---

# JavaScript-Promise.all源码

​		Promise，可以说是前端 js 面试里面的高频考点了。上次面试就让我手撕了一个 Promise.all 方法的源码

​		这里，我先给出promise all方法的要求

```
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
```

* 首先，大概就是一个可迭代的参数传递，返回一个promise的实例。对于所有都all时，返回resolve，对于失败，返回rejected。

```
Promise.all = function(promises) {
  // 因为是可迭代的，同时参数可以是set，map等，所以我们可以先使用 ... 进行全部划分为数组
  const requests = [...promises]

  // 同时因为返回是 promise，所以
  return new Promise((resolve, reject) => {
    
  })
}
```

* 其次，需要进行依次的请求，在 res push进返回的resolve数组，err，直接reject回去。

```
Promise.all = function(promises) {
  // 因为是可迭代的，同时参数可以是set，map等，所以我们可以先使用 ... 进行全部划分为数组
  const requests = [...promises]
  const resArr = []

  // 同时因为返回是 promise，所以
  return new Promise((resolve, reject) => {
  	// 依次进行循环，操作
    for (let i = 0; i < requests.length; i++) {
      const request = requests[i]
      // 判断为 then 还是 catch
      request.then(res => {
      	// res，push进数组
        resArr.push(res)
      }, err => {
      	// catch 直接 rejected
        reject(err)
      })
    }
  })
}
```

​		这里，我们有了一个初步的，然后就是需要判断如果push全部都结束了，那么就应该resolve数组了。

```
count++;
resArr.push(res)
if (count === requests.length) resolve(resArr)
```

​		这里也没有什么，就是进行push，然后判断是否push结束了。

​		写到这里，突然发现一个问题，那就是，不能使用 push 啊，因为异步的不确定性，需要使用下标进行赋值，不然返回的结果会是乱的。

```
Promise.all = function(promises) {
  // 因为是可迭代的，同时参数可以是set，map等，所以我们可以先使用 ... 进行全部划分为数组
  const requests = [...promises]
  const resArr = []
  let count = 0;

  // 同时因为返回是 promise，所以
  return new Promise((resolve, reject) => {
    for (let i = 0; i < requests.length; i++) {
      const request = requests[i]
      request.then(res => {
        count++;
        // resArr.push(res)
        resArr[i] = res
        if (count === requests.length) resolve(resArr)
      }, err => {
        reject(err)
      })
    }
  })
}
```

​		到了这里，感觉快差不多了，但是我们还需要对传递的参数进行一个判断，对于不是Promise的，我们需要直接resolve成功。

​		创建一个 promise判断的函数。这里有两个思路。

* 判断 存在req，同时 then 为function
* 判断 req为对象，同时 then为function。

当然，最后我选择了第一个，因为判断req为对象，需要使用Object.prototype.toString 方法，因为 typeof 对 null 为 object，这里就需要进行额外思考，不然会 req.then 报错。

```
  function isPromise(req) {
    // req 是一个对象，同时then是一个方法。
    if (req && typeof req.then === 'function') {
      return true
    }
    return false
  }
```

​		创建一个添加进入数组的函数

​	这里，我们将resArr[i] 直接复制为 res。可以避免异步产生返回结果不同步出现的问题。

​	同时我们是通过返回一个 bool 来判断运行是否应该结束，当然。这个问题在于我们在resolve之后，可能是最后一个resolve，所以需要再resolve出去，

​	这里有两个方法

* 第一个就是我现在的写法，通过返回bool，判断是否结束，然后在另一个地方进行resolve 处理
* 第二个就是利用了闭包的效果，我们将函数写在了 return new Promise(() => {}) 里面，这样就可以使用闭包，我们只需要在函数里面进行判断是否应该resolve，而不是在外面添加一个判断。

```
  // 进行添加到resArr数组里面，同时判断是否为最后一个
  function requestResolve(res, i) {
    // count++;
    resArr[i] = res
    return ++count === requests.length;
  }
```

​		其实，这里我也不好决定使用哪一种，因为我的代码的切割水平不到位，不知道这个 resolve，应该放在 new promise里面，还是说放在方法内部，主要是放在方法内部，给我一种逻辑被划开了的感觉。

```
Promise.all = function(promises) {
  // 因为是可迭代的，同时参数可以是set，map等，所以我们可以先使用 ... 进行全部划分为数组
  const requests = [...promises]
  const resArr = []
  let count = 0;

  function isPromise(req) {
    // req 是一个对象，同时then是一个方法。
    if (typeof req === 'object'
        && typeof req.then === 'function') {
      return true
    }
    return false
  }

  // 进行添加到resArr数组里面，同时判断是否为最后一个
  function requestResolve(res, i) {
    // count++;
    resArr[i] = res
    return ++count === requests.length;
  }

  // 同时因为返回是 promise，所以
  return new Promise((resolve, reject) => {
    for (let i = 0; i < requests.length; i++) {
      const request = requests[i]
      if (isPromise(request)) {
        request.then(res => {
        	// 这里我是将其分割开来，感觉这样逻辑是分开的。
          requestResolve(res, i) && resolve(resArr)
        }, err => {
          reject(err)
        })
        // err 可以直接使用 reject
        // req.then(res => {}, reject)。
        // 高级，看了别人的代码就是不一样。
      } else {
        requestResolve(request, i) && resolve(resArr)
      }
    }
  })
}
```





