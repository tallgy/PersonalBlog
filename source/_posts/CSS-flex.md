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

​		flex就是弹性布局。

​		基本使用

```
display: flex;
```



## 主轴

​		`flex-direction` 定义主轴

```
row，row-reverse
代表inline，横向延申
```

```
column，column-reverse
代表black，竖向排列
```

​		其中 `reverse `代表翻转，就是反过来的排列顺序



## 交叉轴，副轴

​		就是和主轴垂直的轴

​		如果主轴是横轴，那么交叉轴就是数轴。



# 当使用了 Flex容器

​		在定义了 `display: flex;` 之后的一些默认行为

```
元素排列一行，(因为 flex-direction 的默认值是 row )
从主轴的起始线开始，这里起始线一般就是如果是row，代表了从左向右，其他的	情况查看MDN文档
元素不会再主维度拉伸，但是可以缩小
元素被拉伸来填充交叉轴大小
flex-basis 为 auto
flex-wrap 为 nowrap
```



# 对于flex属性值的行为

## flex-direction 设置主轴

​		对定义了 `display: flex;` 的元素使用

```
flex-direction
	row
		横向，从左到右
	row-reverse
		横向，从右到左
		对于行内块元素和话，宽度的计算是和 Colum-reverse 类似的。
	column
		纵向，从上到下
	column-reverse
		纵向，从下到上
		对于没有设置高度的，就会按照最小高度进行计算，(意思就是说，高度和column的一样，只是方向反了而已。)
```



## flex-wrap 实现多行flex

​		对定义了 `display: flex;` 的元素使用

​		在这样做的时候，您应该把每一行看作一个新的`flex`容器。 任何空间分布都将在该行上发生，而不影响该空间分布的其他行。

```
wrap
	会换行，而不是进行缩小
nowrap
	不会换行，会使用缩小的规则进行缩小，对于不能缩小的，会导致溢出。
```

**注意：**

* 如果在使用 wrap 时，使用了 flex:1， 时，这个时候就需要注意要不要定义 flex-basis 了。

​			**原因**： 因为 `flex: 1`; 对应了 `flex-grow: 1` & `flex-shrink: 1` && `flex-basis: 0`  ，所以其实原因就是 flex-basis 为 0 了。所以如果不自己定义一个 flex-basis 的话，就不会进行换行。

* 还有就是，对于每一个换行之后的元素，每一行都算是一个弹性行，所以对于以下代码

  * ```
    flex: 1 1 160px;
    
    子元素在160px，换行之后，会对每一行的每个元素进行 扩张 和 收缩 的重新计算。
    ```

  * ```
    参考 MDN 链接：
    https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout/Mastering_Wrapping_of_Flex_Items
    ```



## 简写属性 flex-flow

​		对定义了 `display: flex;` 的元素使用

​		是 `flex-direction` 和 `flex-wrap` 的组合

​		对于没有写的，就会使用默认值：row 和 nowrap

**注：**

* 从谷歌浏览器的显示来说:
  * 如果设置 `flex-flow: column; `  ，就直接代表了 `flex-direction: column;` ，而不会定义 `flex-wrap`
  * 如果设置 `flex-flow: wrap; `  ，就代表了 `flex-direction: initial;` 而  `flex-flow: wrap; ` 



## flex-basis 元素的空间大小

​		该属性的默认值是 `auto` ，此时，浏览器会检查这个元素是否具有确定的尺寸(width / heght ，这个看是使用 row 还是 Colum)，如果**具有确定的尺寸**，就会将该值设置为 flex-basis。如果没有设定尺寸，就会采用**元素内容的尺寸**。如果不使用  [`box-sizing`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-sizing) 改变盒模型的话，那么这个属性就决定了 flex 元素的内容盒（content-box）的尺寸。



## flex-grow 延展比例

​		负值无效，默认为0

​		简单来说，这个是一个比例，对于存在可用空间的(可用空间：就是指在使用之后，父元素还存在剩余的空间。)， 子元素会根据这个比例将可用空间占据。

​		比如：两个子元素，一个为 1， 一个为 2，父元素的可用空间为 90，那么第一个就会扩张 1/(1+2) * 90 ， 第二个就会扩张 2/(1+2) * 90 。



## flex-shrink 收缩比例

​		负值无效，默认为1

​		简单来说，就会对于如果容器不够排列 flex元素的空间。那么就会按照比例进行收缩。默认为1

​		计算方式： 这里的计算方式略和 `flex-grow` 有所不同。他的计算会和 `flex-basis` 相关联。这里给出计算的方法

​			每个元素的 `flex-basis` * `flex-shrink` 这个值作为一个比例**X1**，将所有参与计算的元素进行计算 **X1~Xi 的求和 Xs** ，然后再计算出超出的量，这个超出的量是 **overflow = 子元素的flex-basis 的总和 - 父元素的宽度**， 然后就是进行计算 **Xi / Xs * overflow**， 这个计算出的值就是需要减去的量，这里可能有点抽象，我们也给一个例子方便理解

```
div	width:600px
	div flex: 1 1 200px;
	div flex: 1 1 200px;
	div flex: 1 2 300px;

首先我们可以看出，这个是超出了，所以需要收缩，
超出的量 overflow = 600 - 200 - 200 - 300 = 100px
然后比例的计算
	X1 = 1 * 200 = 200px
	X2 = 1 * 200 = 200px
	X3 = 2 * 300 = 600px
所以 Xs = X1 + X2 + X3 = 1000px
然后就是计算收缩的值	Xi / Xs * overflow
	200 / 1000 * 100 = 20
	200 / 1000 * 100 = 20
	600 / 1000 * 100 = 60
	
所以最终的宽度是
div width: 600px
	div width: 180px
	div width: 180px
	div width: 240px
```

在这里我也看到了一个更详细的一些计算教程，虽然里面的内容是否正确我没有进行参考，但是写的很详细，我就把链接贴出来

```
https://www.cnblogs.com/liyan-web/p/11217330.html
```

​	**注意点：**

```
https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout/Controlling_Ratios_of_Flex_Items_Along_the_Main_Ax
```

​		这个元素在缩小的时候，不会小于最小宽度，至于这个最小宽度定义，我现在认为就是可以造成溢出的情况。举个例子：

```
div
	222222222222222222222222222222222222
	
	在这个情况，如果这个div想对于父元素的弹性布局，造成了溢出，但是因为最小的宽度为这个文字的一个长度，不会换行，所以会造成溢出。
	而 22222 222222 222222222 222222 2222，这里，因为在 文字里面加入了空格，在html文档里面一个最小宽度是一个空格隔开的一个距离，所以会出现换行而不会让宽度溢出的情况。
	这里记录一下会统计最小宽度的情况。
		' '(不能使用 &nbsp; ), '。'(。会在上一行), '，'(中文，号)，这里的原因应该是中文的。和， 里面的一个字符形势和 . ,不一样。
	注意：只能是 英文和数字，中文会自动换行。原因应该是 对于英文，如果不打空格，浏览器会认为是一个单词，所以不会换行。
```

**注意：**

* `flex-grow` && `flex-shrink` 的值要为正数。




## 简写属性 flex

​		对弹性布局的子元素使用。

​		包含了： `flex-grow`  、 `flex-shrink` 和 `flex-basis` 

​		对于每个属性的值，都有理解了。我们就说几个简写的代表意思

- `flex: initial`
  - 代表初始值：`flex: 0 1 auto`
- `flex: auto`
  - 代表 `flex: 1 1 auto`
- `flex: none`
  - 代表 `flex: 0 0 auto`
- `flex: <positive-number>`
  - 代表 `flex: x x 0`



## align-items 交叉轴方向对齐

​		默认为 `stretch `，默认会被拉伸到最高元素的高度。

```
stretch
	拉伸到最高元素的高度
flex-start
	按flex容器的顶部对齐
flex-end
	按flex容器的下部对齐
center
	使它们居中对齐
```



## justify-content 主轴方向对齐

​		初始值是 `flex-start`， 元素从容器的起始线排列。

```
stretch
	不了解
flex-start
	从容器的起始线排列
flex-end
	从容器的终止线开始排列
center
	在中间排列
space-around
	把元素排列好之后的剩余空间拿出来，平均分配到元素之间，所以元素之间间隔相等
space-between
	使每个元素的左右空间相等
```



# 对于flex的溢出：

### 方式一

```
对一个元素设置了flex布局，其中的子元素的子元素的宽度可以造成溢出
```

```
div.father
	div.children
	
father 元素设置了弹性布局， children 元素只设置一个宽度，会发现 children 的宽度不会高于father的宽度。主要是因为涉及了
```

<img src="CSS-flex/image-20211027210947232.png" alt="image-20211027210947232" style="zoom: 50%;" />



```
但是
div.father
	div.children
		div.cChildren

这个对于 cChildren 的宽度就会将 children 的宽度顶出来，造成溢出。
其中，这里的宽度，如果 cChildren 大于了 father 就会将 cChildren 和 children 顶出来。
但是如果 cChildren 大于了 children 的话，就会再 children 到达他所定义的宽度之后，就不会继续扩张了。
```

<img src="CSS-flex/image-20211027211015004.png" alt="image-20211027211015004" style="zoom:50%;" />



<img src="CSS-flex/image-20211027212431340.png" alt="image-20211027211015004" style="zoom:50%;" />



​		但是这里的计算方式我没有理解到为什么。



### 方式二

​		设置 `flex-shrink: 0;` 属性，因为设置了0之后代表了不会对超过的部分进行处理。



### 方式三

​		这个是文字超出导致的溢出，就是如果文字，或者 img图片的设置超过了宽高也会造成溢出。

​		这个的原因是因为，对于英文和数字，浏览器不能识别你的单词是否是一个，所以如果你输入一串英文和数字，中间没有分割(前面有说)，就会造成溢出，而汉字不行。

<img src="CSS-flex/image-20211027213217557.png" alt="image-20211027213217557" style="zoom:50%;" />



### 方式四

​		我们可以发现，flex 对于溢出的处理，默认只是针对于主轴，所以对于侧轴是没有溢出处理的。

```
.father {
    display: flex;
    flex-direction: column;
    width: 50px;
}

.children {
	width: 60px;
}
```

<img src="CSS-flex/image-20211027214029980.png" alt="image-20211027214029980" style="zoom:50%;" />



