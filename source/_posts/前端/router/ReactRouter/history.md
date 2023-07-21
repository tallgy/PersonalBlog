---
title: history
date: 2023-07-20 17:49:08
tags:
 - React
categories:
 - React
 - Router
 - history
---


# GitHub 

https://github.com/remix-run/history


# history

里面主要是涉及了几个方法

## 3 个创建 history 方法

这三个方法主要区别在于存储 url 的方式不同
但是 history 内部将其 api 封装统一了起来，方便其使用

* createBrowserHistory
  * 浏览器 url 模式
* createHashHistory
  * 浏览器 hash 模式
* createMemoryHistory
  * 存储在内存，常用于没有 url 的展示时


## api

### get action

```typescript

export enum Action {
  /**
   * 他表示是对 history 的任意索引更改，比如使用 go back 等，没有描述导航的方向，只是索引的变化
   * 注意：这是新建对象的默认操作
   */
  Pop = "POP",

  /**
   * PUSH表示要添加到历史堆栈中的新条目，例如单击链接和加载新页面时。
   * 发生这种情况时，堆栈中的所有后续条目都将丢失。
   * 这句话的意思是，比如我当前并不是最顶层的 history（比如使用 go back 回去过），
   * 然后我使用了 push 之后，我会让后面的 stack 都丢失
   */
  Push = "PUSH",

  /**
   * REPLACE表示历史堆栈中当前索引处的条目正在被新条目替换。
   */
  Replace = "REPLACE",
}

```

代表当前 url 是用什么模式进来的 pop、push、replace
默认是 pop
在当我们调用 push、replace 方法的时候，更新对应的值。


### get location


```typescript


/**
 * URL路径名，以/开头。
 */
export type Pathname = string;

/**
 * URL搜索字符串，以？开头？。
 */
export type Search = string;

/**
 * URL片段标识符，以#开头。
 */
export type Hash = string;

/**
 * 一个对象，用于将某些任意数据与位置相关联，但不出现在URL路径中。
 */
export type State = unknown;

/**
 * 与位置关联的唯一字符串。可用于在其他一些存储API中安全存储和检索数据，如“localStorage”。
 */
export type Key = string;

/**
 * URL的路径名、搜索和哈希值。
 */
export interface Path {
  pathname: Pathname;
  search: Search;
  hash: Hash;
}

/**
 * 历史堆栈中的一个条目。位置包含有关URL路径的信息，以及可能的一些任意状态和密钥。
 */
export interface Location extends Path {
  /**
   * 与此位置关联的任意数据的值。
   */
  state: unknown;

  /**
   * 注意：此值在初始位置始终为“默认值”。
   */
  key: Key;
}


```


是一个当前的 location 信息，比如当前 url 的数据，
以及我们将一些数据存储在 history 中的 state 时

历史堆栈中的一个条目。位置包含有关URL路径的信息，以及可能的一些任意状态和密钥。


### createHref

创建链接，说白了就是根据传参来得到url。


### push

简述就是 先判断 blocks 是否通过，
然后 调用 pushState 进行跳转
最后调用 listen 进行监听


```JavaScript

function push(to: To, state?: any) {
  let nextAction = Action.Push;
  // 生成一个 location 里面主要是，对 key进行了随机生成
  // 用 freeze 冻结对象属性
  let nextLocation = getNextLocation(to, state);

  // 重试方法，在 blocks 里面进行调用
  function retry() {
    push(to, state);
  }

  // 代表执行了 blocks 里面之后允许了跳转
  if (allowTx(nextAction, nextLocation, retry)) {
    let [historyState, url] = getHistoryStateAndUrl(nextLocation, index + 1);

    // TODO: Support forced reloading
    // try...catch because iOS limits us to 100 pushState calls :/
    // 调用 window 自带的 pushState。
    try {
      globalHistory.pushState(historyState, "", url);
    } catch (error) {
      // They are going to lose state here, but there is no real
      // way to warn them about it since the page will refresh...
      window.location.assign(url);
    }

    // 调用 listen 监听
    applyTx(nextAction);
  }
}

```


### replace

和 push 差不多一样的逻辑
使用 window.history.replaceState 方法进行调用


### go


```JavaScript

// 代码十分简单，就是调用一下 window.history.go 方法
function go(delta: number) {
  globalHistory.go(delta);
}

```


### back

调用 go(-1)


### forward

调用 go(1)


### listen

主要是监听了 history的变化，
listen 仅仅只做监听使用

主要是几个函数


createEvents

length 返回监听的数量
push push 一个监听的方法，会返回一个 unlisten 方法，本质就是 把监听数组过滤掉
call forEach 循环调用 方法

```JavaScript

function createEvents<F extends Function>(): Events<F> {
  let handlers: F[] = [];

  return {
    get length() {
      return handlers.length;
    },
    push(fn: F) {
      handlers.push(fn);
      return function () {
        handlers = handlers.filter((handler) => handler !== fn);
      };
    },
    call(arg) {
      handlers.forEach((fn) => fn && fn(arg));
    },
  };
}

```


listen 本质原理

```JavaScript

// 先通过 listen 添加到监听数组里面
listen(listener) {
  return listeners.push(listener);
},

// 在 applyTx 里面进行调用
function applyTx(nextAction: Action) {
  action = nextAction;
  [index, location] = getIndexAndLocation();
  listeners.call({ action, location });
}

// applyTx 会在 handlePop、push、replace 中调用
// push、replace 简单易理解
// handlePop 表明的是如果在历史的两个 history 导航中进行跳转就会触发。
```


### block


用于阻塞路由跳转

```JavaScript


block(blocker) {
  // 和 listen 的 createEvents 是同一个
  let unblock = blockers.push(blocker);

  // 这里要用 === 1，目的是为了，只添加一次
  // 这个的作用是，如果用户不是通过正常的 push、replace 进行的操作
  // 那么我们就不能使用回调函数，所以，我们就会通过使用监听去进行默认的阻塞行为。。
  // 这个 promptBeforeUnload 函数的作用就是，进行默认的阻塞行为，让其不进行跳转
  if (blockers.length === 1) {
    window.addEventListener(BeforeUnloadEventType, promptBeforeUnload);
  }

  return function () {
    // 取消 block 的监听
    unblock();

    // See https://html.spec.whatwg.org/#unloading-documents
    // 删除beforeunload侦听器，以便文档在pagehide事件中仍然可用。
    // promptBeforeUnload 里面会有 event preventDefault 进行事件拦截，所以需要移除。
    if (!blockers.length) {
      window.removeEventListener(BeforeUnloadEventType, promptBeforeUnload);
    }
  };
},

// block 方法主要依赖于  allowTx 和 handlePop
function allowTx(action: Action, location: Location, retry: () => void) {
  return (
    // 目的是，如果 blocker存在，那么就会去执行 call 方法，
    // 但是此时就需要 return false，所以使用了 const b = (a, false); 这种逻辑
    !blockers.length || (blockers.call({ action, location, retry }), false)
  );
}
// handlePop 主要是进行 刷新 go 方法等调用会触发
// allowTx 主要在 push、replace 中会调用
// 核心就是这样，如果 allowTx 为 true 才会去push，否则会造成阻塞
// 依赖于 回调函数中去调用 retry 来进行跳转。
function push(to: To, state?: any) {
  if (allowTx(nextAction, nextLocation, retry)) {
    globalHistory.pushState(historyState, "", url);
    applyTx(nextAction);
  }
}
```


## createBrowserHistory createHashHistory createMemoryHistory 三个具体的实现区别

首先我们知道三个分别代表了不同的url的表示方式

### createBrowserHistory 和 createHashHistory 

两个的主要区别在于一个是使用了 hash 来存储 url，另一个是直接使用了url。
所以区别就会体现在

#### 对于 url 的信息获取不同

hash 会通过一个 parsePath 来进行一次转义

```JavaScript
  
/**
 * 将字符串URL路径解析为其单独的路径名、搜索和散列组件。
 */
export function parsePath(path: string): Partial<Path> {
  let parsedPath: Partial<Path> = {};

  if (path) {
    let hashIndex = path.indexOf("#");
    if (hashIndex >= 0) {
      parsedPath.hash = path.substr(hashIndex);
      path = path.substr(0, hashIndex);
    }

    let searchIndex = path.indexOf("?");
    if (searchIndex >= 0) {
      parsedPath.search = path.substr(searchIndex);
      path = path.substr(0, searchIndex);
    }

    if (path) {
      parsedPath.pathname = path;
    }
  }

  return parsedPath;
}

```