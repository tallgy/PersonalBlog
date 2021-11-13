---
title: CSS伪类和伪元素
date: 2021-11-13 23:08:34
tags:
 - CSS
 - 随笔
categories:
 - CSS
 - 随笔
---



#  CSS伪类和伪元素

```
https://developer.mozilla.org/zh-CN/docs/Learn/CSS/Building_blocks/Selectors/Pseudo-classes_and_pseudo-elements
```



## 伪类

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-classes
```

​		伪类是选择器的一种，它用于选择处于特定状态的元素，比如当它们是这一类型的第一个元素时，或者是当鼠标指针悬浮在元素上面的时候。

```
使用的是单冒号
:
```

### 行为伪类

​		一些伪类只会在用户以某种方式和文档交互的时候应用。这些**用户行为伪类**，有时叫做**动态伪类**，表现得就像是一个类在用户和元素交互的时候加到了元素上一样。

```
:hover,:focus
```

```
div:hover {}
```



## 伪元素

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-elements
```

​		伪元素以类似方式表现，不过表现得是像你往标记文本中加入全新的HTML元素一样，而不是向现有的元素上应用类。伪元素开头为双冒号`::`。

> **备注：**一些早期的伪元素曾使用单冒号的语法，所以你可能会在代码或者示例中看到。现代的浏览器为了保持后向兼容，支持早期的带有单双冒号语法的伪元素。

```
::before
::after
```

```
//before和after的伪元素 content是必须的。
div::before {
	content: '';
}
```



# 区别

* 伪元素是生成一个新的，或者表现的像是生成了一个新的HTML元素。

  伪类是选择处于某种状态的的元素。

* 伪元素是使用的 ::

  伪类是使用的 :

