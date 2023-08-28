---
title: performance浏览器性能API
date: 2023-08-28 18:10:30
tags:
 - performance
categories:
 - performance
---

# performance

* 性能接口 API 
* Performance 接口可以获取到当前页面中与性能相关的信息。它是 High Resolution Time API 的一部分，同时也融合了 Performance Timeline API、Navigation Timing API、User Timing API 和 Resource Timing API。
* 该类型的对象可以通过调用只读属性 Window.performance 来获得。


## API



### Performance.clearMarks()

将给定的 mark 从浏览器的性能输入缓冲区中移除。


### Performance.clearMeasures()

将给定的 measure 从浏览器的性能输入缓冲区中移除。


### Performance.clearResourceTimings() (en-US)

从浏览器的性能数据缓冲区中移除所有 entryType 是 "resource" 的 performance entries。


### Performance.getEntries()

基于给定的 filter 返回一个 PerformanceEntry 对象的列表。


### Performance.getEntriesByName()

基于给定的 name 和 entry type 返回一个 PerformanceEntry 对象的列表。


### Performance.getEntriesByType() (en-US)

基于给定的 entry type 返回一个 PerformanceEntry 对象的列表


### Performance.mark()

根据给出 name 值，在浏览器的性能输入缓冲区中创建一个相关的timestamp


### Performance.measure()

在浏览器的指定 start mark 和 end mark 间的性能输入缓冲区中创建一个指定的 timestamp


### Performance.now()

返回一个表示从性能测量时刻开始经过的毫秒数 DOMHighResTimeStamp


### Performance.setResourceTimingBufferSize() (en-US)

将浏览器的资源 timing 缓冲区的大小设置为 "resource" type performance entry 对象的指定数量


### Performance.toJSON()

其是一个 JSON 格式转化器，返回 Performance 对象的 JSON 对象




# 引用
```
https://developer.mozilla.org/zh-CN/docs/Web/API/Performance
```
