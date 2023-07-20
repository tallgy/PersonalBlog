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

# history

```
https://github.com/remix-run/history
```

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




### replace




### go




### back




### forward




### listen




### block







