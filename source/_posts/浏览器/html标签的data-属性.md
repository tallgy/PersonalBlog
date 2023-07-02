---
title: html标签的data-*属性
date: 2022-02-02 19:25:12
tags:
 - HTML
categories:
 - HTML
---



#  什么是HTML标签的 data-* 属性

​		通过对于图片懒加载的学习和requireJs中的一个了解。我们发现了一个处于标签中的一个属性data-*。那么这个属性是什么意思呢。



​		首先通过了解。我们知道了，data-*，这个属性是属于HTML5的一个新的特性。特点就是，可以通过这个方式给HTML标签添加新的属性。



## 获取方式

### 使用 getAttribute进行获取

```
elem.getAttribute( name );
```



### 使用 HTML5 欣特性 dataset 进行获取

```
elem.dataset.name 注意，使用 短横线命名的需要改为驼峰来进行取值。
```



