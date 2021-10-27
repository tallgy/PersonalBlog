---
title: CSS-flex
date: 2021-10-27 15:54:35
tags:
 - CSS
 - flex
categories:
 - CSS
---



#  CSS-flex布局

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox
```

flex就是弹性布局。

基本使用

```
display: flex;
```



## 主轴

`flex-direction` 定义主轴

```
row，row-reverse
代表inline，横向延申
```

```
column，column-reverse
代表black，竖向排列
```

其中 `reverse `代表翻转，就是反过来的排列顺序



## 交叉轴，副轴

就是和主轴垂直的轴

如果主轴是横轴，那么交叉轴就是数轴。



# 当使用了 Flex容器

在定义了 `display: flex;` 之后的一些默认行为

```
元素排列一行，(因为 flex-direction 的默认值是 row )
从主轴的起始线开始，这里起始线一般就是如果是row，代表了从左向右，其他的	情况查看MDN文档
元素不会再主维度拉伸，但是可以缩小
元素被拉伸来填充交叉轴大小
flex-basis 为 auto
flex-wrap 为 nowrap
```

