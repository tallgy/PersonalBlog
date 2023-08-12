---
title: rem
date: 2021-10-16 14:14:53
tags: 
 - CSS
 - 长度单位
categories:
 - CSS
---


MDN 参考

```
https://developer.mozilla.org/zh-CN/docs/Learn/CSS/Building_blocks/Values_and_units
```



# rem 和 em

## em

再说rem之前，先说一下em

```
	在 font-size 中使用是相对于父元素的字体大小，在其他属性中使用是相对于自身的字体大小，如 width
```

## rem

```
	rem 就是 root em
	根元素的字体大小
```

rem是相对于根元素html的字号进行了等比例的计算。

for example

```
html { font-size: 16px; }

ohter { width: 300/16 rem; }
```

## rem使用方式

​		简单来说，就是从px转化成了一个rem，所以为了方便计算，常用的方式就是 1rem 转化成 10px 或者 100px。

​		所以对于一个 750px 的设计图，就将其除以 75或者7.5，` font-size=10px | 100px` 



# vh 和 vw

简单来说就是相对于视窗宽度和高度的百分比，会计算出视窗的高度和宽度

```
vw：视窗宽度的1%
vh：视窗高度的1%
```

如果将浏览器缩小了，对应的百分比值也会缩小。



# vmin 和 vmax

```
vmin：视窗较小尺寸的1%
vmax：视图大尺寸的1%
```

大致和上面的那个一样一个意思。



# 百分比

​		百分比简单来说，就是相对于父级，但是对于边距是不一样的方式。

```
	如果将元素的字体大小设置为百分比，那么它将是元素父元素字体大小的百分比。如果使用百分比作为宽度值，那么它将是父值宽度的百分比。
```

