---
title: CSS-元素选择器
date: 2021-10-25 19:51:58
tags:
 - CSS
 - 元素选择器
categories:
 - CSS
 - 元素选择器
---



#  CSS-元素选择器



## 相邻兄弟选择器

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Adjacent_sibling_combinator
```

​		**相邻兄弟选择器** (`+`) 介于两个选择器之间，当第二个元素*紧跟在*第一个元素之后，并且两个元素都是属于同一个父`元素`的子元素，则第二个元素将被选中。

**注：**

​	第一，要相邻之后的，不相邻的，在前面的不管。 

​	只会选择一个，就算第二个元素有多个满足，只会选择最开始的一个。

​	但是对于第一个元素就可以有很多个。

```
.f + .b {
	...
}
```



## 属性选择器

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Attribute_selectors
```

​		CSS **属性选择器**通过已经存在的属性名或属性值匹配元素。

选择存在这个属性的元素，这个属性不一定要含有意义，也不一定有值

```
div[cc] {
	...
}

存在cc，但是cc是一个没有意义的属性，也可以被选中。
<div class="b" cc>1</div>
```



### 属性选择器值的匹配

```
存在属性cc，并且值为xxx
[cc='xxx']

<div class="b" cc="xxx">1</div>
```

```
存在属性cc，并且值包含了xx
cc *= 'xx'

<div class="b" cc="111xxasf">1</div>
```

```
存在属性cc，并且该属性是一个以空格作为分隔的值列表，其中至少有一个值为 xx。
cc ~= 'xx'

<div class="b" cc="111 xx asf">1</div>
```

```
存在属性cc，并且属性值为“xx”或是以“xx-”为前缀
cc |= 'xx'

<div class="b" cc="xx-asf">1</div>
```

```
存在属性cc，并且属性值是以 xx 开头的元素。
cc ^= 'xx'

<div class="b" cc="xxasf">1</div>
```

```
存在属性cc，并且属性值是以 xx 结尾的元素。
cc $= 'xx'

<div class="b" cc="awwxx">1</div>
```

```
存在属性cc，并且属性值是以 xx 结尾的元素。
cc $= 'xx'

<div class="b" cc="awwxx">1</div>
```

```
在属性选择器右方括号前加 i用空格隔开，表示忽略大小写
[cc $= 'xx' i]

div[cc $= 'xx' i] {
	...
}
```

```
同上，使用 s，表示区分大小写
```



## 子选择器

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Child_combinator
```

​		当使用  `>` 选择符分隔两个元素时,它只会匹配那些作为第一个元素的**直接后代(**子元素)的第二元素. 

**重点是 直接后代，而不是孙子代**

```
ele1 > ele2 {
	...
}
```



## 类选择器

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Class_selectors
```

​		在一个HTML文档中，CSS类选择器会根据元素的类属性中的内容匹配元素。类属性被定义为一个以空格分隔的列表项，在这组类名中，必须有一项与类选择器中的类名完全匹配，此条样式声明才会生效。

​	简单来说，就是class的属性，值是以空格进行的分割，需要其中一个值满足类选择器的属性值，才能有作用。



```
.class { ... }

他和属性选择器的具有相同的作用。
[class ~= 'class'] { ... }
```



## 后代选择器

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Descendant_combinator
```

​		**后代组合器**（通常用单个空格（` `）字符表示）组合了两个选择器，如果第二个选择器匹配的元素具有与第一个选择器匹配的祖先（父母，父母的父母，父母的父母的父母等）元素，则它们将被选择。利用后代组合器的选择器称为后代选择器。

​	简单来说，就是，第二个元素是第一个元素的后代，但是可以不是直接后代。

```
.a .c { ... }

<div class="a">
  <div>
    <div class="c"></div>
  </div>
</div>
如同上面这样，a存在后代c，但是不是直接后代，可以使用后代选择器(` `), 而不能使用子选择器(`>`)
```



## 通用兄弟选择器

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/General_sibling_combinator
```

​		兄弟选择符，位置无须紧邻，只须同层级，`A~B` 选择`A`元素之后所有同层级`B`元素。

```
ele1 ~ ele2 { ... }
```



## ID 选择器

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/ID_selectors
```

​		CSS ID 选择器会根据该元素的 ID 属性中的内容匹配元素,元素 ID 属性名必须与选择器中的 ID 属性名完全匹配，此条样式声明才会生效。

**注：**

​	一般一个html里面，一个id只能一个，如果有多个，那么这个就会选中多个。

​	id属性的值只有一个，不像使用class属性那样，可以有空格划分

``` 
#id { ... }

同属性选择器的
[id=value] { ... }
```



## 选择器列表

选择器分组

​		CSS **选择器列表**（`,`），常被称为并集选择器或并集组合器，选择所有能被列表中的任意一个选择器选中的节点。

```
h1,
#id,
.class,
.a > span {
	...
}
```

**注意点：**

​		选择器列表无效化，说的是当一个选择器不被支持，就会出现整条规则全部失效，我这里看了一个人写的，[出现无效的伪选择器](https://www.xinran001.com/frontend/248.html) 他的说明是如果是一个伪选择器无效的话，就会出现这个问题。但是如果是一个选择器的写法满足一个浏览器的cssom的构建，那么就不会出现这个问题。

​			解决这个问题的一个方法是使用 [`:is()`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:is) 选择器，它会忽视它的参数列表中失效的选择器，但是由于 [`:is()`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:is) 会影响优先级的计算方式，这么做的代价是，其中的所有选择器都会拥有相同的优先级。



## 元素选择器

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Type_selectors
```

​		CSS元素选择器(也称为类型选择器)通过node节点名称匹配元素.

```
ele { ... }

span { ... }
```



## 通配选择器

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Universal_selectors
```

​		在CSS中,一个星号(`*`)就是一个通配选择器.它可以匹配任意类型的HTML元素

```
* { ... }
```

​		CSS3里面，*可以和命名空间搭配使用，命名空间，我现在也没有了解，我们后面在学。

```
ns|* - 会匹配ns命名空间下的所有元素
*|* - 会匹配所有命名空间下的所有元素
|* - 会匹配所有没有命名空间的元素
```



## 伪类选择器

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-classes
```

### 常用的

**active**

```
a:active 
可以代表按下按键和松开按键。常用于按钮和链接
```

**disabled**

```
表示被禁用的元素
```

**enabled**

```
没有被禁用的元素
```

**first-child**

```
一组兄弟元素中的第一个元素。

p:first-child，代表的是p的第一个
```

**focus**

```
获得焦点时
```

**hover**

```
鼠标放在上面时触发
```

MDN上有个描述

```
	:hover CSS伪类适用于用户使用指示设备虚指一个元素（没有激活它）的情况。这个样式会被任何与链接相关的伪类重写，像:link, :visited, 和 :active等。为了确保生效，:hover规则需要放在:link和:visited规则之后，但是在:active规则之前，按照LVHA的循顺序声明:link－:visited－:hover－:active。
```

没有理解什么意思。

**invalid**

```
:invalid CSS 伪类 表示任意内容未通过验证的 <input> 或其他 <form> 元素 .
```

**last-child**

```
一组兄弟元素中的最后一个元素。
```

**link**

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/:link
看看
```

**not**

```
反选

:not(p)
不要p标签的
```

**nth-child**

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/:nth-child
```

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-classes
```

