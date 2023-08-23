---
title: qiankun
date: 2023-08-23 15:36:44
tags:
 - qiankun
categories:
 - qiankun
---

# qiankun 微前端



## scope css

原理比较简单

就是使用 cssRule 将 textContent 添加上 scope
首先因为微应用的设置，所以顶部会有一个 div 包裹，和一个 data-qiankun 的 appName
所以 结合起来将 css 拼接 添加上了 scope

```JavaScript

const styleNodes = appElement.querySelectorAll('style') || [];
styleNodes.forEach(styleNode => {
  // 详细请看 styleSheet https://developer.mozilla.org/zh-CN/docs/Web/API/StyleSheet
  const sheet = styleNode.sheet as any; // type is missing
  // 详情请看 https://developer.mozilla.org/zh-CN/docs/Web/API/CSSRule
  // arrayify 主要是变为数组效果
  const rules = arrayify<CSSRule>(sheet?.cssRules ?? []);
  // 这个可以让每一个 css 都添加上 scope
  const css = this.rewrite(rules, prefix);
  // eslint-disable-next-line no-param-reassign
  // 覆盖原来的 textContent 可以生效
  styleNode.textContent = css;

  (styleNode as any)[ScopedCSS.ModifiedTag] = true;
})

```


## shadow DOM

比较简单就是，直接创建 shadow DOM 

这里需要注意的是： style 需要放在 shadowDOM 里面，否则不会有作用。
粗略测试

```JavaScript

const { innerHTML } = appElement;
appElement.innerHTML = '';
let shadow;
console.log(appElement, appElement.attachShadow);
shadow = appElement.attachShadow({ mode: 'open' });
shadow.innerHTML = innerHTML;

```


## 沙箱模式 sandbox


