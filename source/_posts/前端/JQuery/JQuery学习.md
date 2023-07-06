---
title: JQuery学习
date: 2022-02-02 19:53:24
tags:
 - JQuery
categories:
 - JQuery
---



#  学习

```
version: 3.6.0
```



g: 4430

```
获取标签属性里面的data自定义属性
function dataAttr( elem, key, data )

这里的 replace 使用的是正则表达式，搭配的 $&, 这里 $& 代表了与 regexp 相匹配的子串。
remutiDash 是 大写字母/g，意思就是说，将大写转为 中划线然后再用 toLowerCase 转为小写。
name = "data-" + key.replace( rmultiDash, "-$&" ).toLowerCase();
```

