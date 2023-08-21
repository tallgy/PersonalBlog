---
title: import-html-entry
date: 2023-08-21 19:59:10
tags:
 - 微前端
 - qiankun
 - import-html-entry
categories:
 - 微前端
---

# import-html-entry

这个里面主要是一个 importEntry 方法
用于将 html 里面的内容 最终分割成 script 、 style html 三个部分
其中貌似有使用类似于 模块化+proxy 的方式处理了js的隔离问题。
css 只是将全部放入了 style 标签中

最终返回的结构是
```JavaScript
res = {
  /** 一份 style、html 内容被取出来 但是 script 没有取出来的string */
  template,
  /** public path */
  assetPublicPath: getPublicPath(entry),
  /** get script 方法、获取 script */
  getExternalScripts: () => getExternalScripts(scripts, fetch),
  /** get style 方法 */
  getExternalStyleSheets: () => getExternalStyleSheets(styles, fetch),
  /** 执行 script */
  execScripts: (proxy, strictGlobal, opts = {}) => {
    if (!scripts.length) {
      return Promise.resolve();
    }
    return execScripts(scripts[scripts.length - 1], scripts, proxy, {
      fetch,
      strictGlobal,
      ...opts,
    });
  },
}

```
