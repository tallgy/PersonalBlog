---
title: single-spa-css
date: 2023-08-19 22:13:41
tags:
 - single-spa-css
categories:
 - single-spa-css
---

# single-spa-css

css 原理本质就是创建 css link DOM
然后对应在
bootstrap 、 mount 、 unmout 方法进行处理 cssUrls 数组的属性


## bootstrap

属于 预加载 css
ref=preload

## mount

在mount也会再次看看是否有没有加载成功的。
同时存储在 linkElements 和 linkElementsToUnmount 属性里面

## unmount

移除 linkElementsToUnmount 里面的数组 dom
同时也会删除对应 linkElements 的。
