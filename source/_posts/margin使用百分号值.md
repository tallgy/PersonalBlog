---
title: margin使用百分号值
date: 2021/10/13
tags: CSS
---



#  CSS margin

```
https://www.w3school.com.cn/cssref/pr_margin-top.asp
```

对于使用 % 为单位的，为父元素的宽度，

```
margin-top: 50%
是相对于父节点的宽度。其他同样
padding-top: 25%;


border: black solid 25%; 使用会报错

对于宽度和高度的50%，就是针对的父元素的宽度和高度了。
```



```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
  <style>
    * {
      margin: 0;
      padding: 0;
    }

    .f {
      height: 400px;
      width: 800px;
      background-color: red;
      /*这里使用 overflow: hidden;
       解决外边距合并，外边距塌陷问题 */
      overflow: hidden;
    }

    .c {
      height: 100px;
      width: 200px;
      background-color: blue;
      margin-top: 25%;
    }

  </style>

</head>
<body>
<div class="f">
  <div class="c"></div>
</div>
</body>
</html>
```

