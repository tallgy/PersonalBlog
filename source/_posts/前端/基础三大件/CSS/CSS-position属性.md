---
title: CSS-position属性
date: 2023-05-26 15:37:16
tags:
categories:
---

# position的属性

```
static 正常
relative 相对定位
absolute 绝对定位
fixed 固定定位
sticky 粘性定位
```

## static
    该关键字指定元素使用正常的布局行为，即元素在文档常规流中当前的布局位置。此时 top, right, bottom, left 和 z-index 属性无效。

## relative

* 该关键字下，元素先放置在未添加定位时的位置，再在不改变页面布局的前提下调整元素位置（因此会在此元素未添加定位时所在位置留下空白）。position:relative 对 table-*-group, table-row, table-column, table-cell, table-caption 元素无效。
* 可以看到当我们设置left时，它相对与自身原本的位置进行了偏移,相对自身定位

## absolute
    
* 元素会被移出正常文档流，并不为元素预留空间，通过指定元素相对于最近的非 static 定位祖先元素的偏移，来确定元素位置。绝对定位的元素可以设置外边距（margins），且不会与其他边距合并。
* 经常使用 子绝父相

## fixed
    
* 元素会被移出正常文档流，并不为元素预留空间，而是通过指定元素相对于屏幕视口（viewport）的位置来指定元素位置。元素的位置在屏幕滚动时不会改变。打印时，元素会出现在的每页的固定位置。fixed 属性会创建新的层叠上下文。当元素祖先的 transform、perspective、filter 或 backdrop-filter 属性非 none 时，容器由视口改为该祖先。

## sticky

* 元素根据正常文档流进行定位，然后相对它的最近滚动祖先（nearest scrolling ancestor）和 containing block（最近块级祖先 nearest block-level ancestor），包括 table-related 元素，基于 top、right、bottom 和 left 的值进行偏移。偏移值不会影响任何其他元素的位置。 该值总是创建一个新的层叠上下文（stacking context）。注意，一个 sticky 元素会“固定”在离它最近的一个拥有“滚动机制”的祖先上（当该祖先的 overflow 是 hidden、scroll、auto 或 overlay 时），即便这个祖先不是最近的真实可滚动祖先。这有效地抑制了任何“sticky”行为（详情见 Github issue on W3C CSSWG）。

### sticky 的简易 JavaScript 代码实现

```JavaScript
// CSS 代码：添加 sticky 和 relative 类

    .relative {
      position: relative;
    }

    .sticky-bottom {
      position: fixed;
      bottom: 0;
    }
    .sticky-top {
      position: fixed;
      top: 0;
    }
    .sticky-left {
      position: fixed;
      left: 0;
    }
    .sticky-right {
      position: fixed;
      right: 0;
    }


// JavaScript 代码：
// 现存的问题，如果使用了 fixed 之后，会出现位置的不同，所以会如果只添加 left 或者 top 等，可能导致
// 元素本来是在屏幕范围里面，但是却看不到的情况

const sticky = (dom) => {
      const {offsetTop, offsetWidth, offsetHeight, offsetLeft} = dom;

      const isOverflow = () => {
          // 滚动高度 超过 偏移高度，上面溢出
        const topOverflow = window.scrollY > offsetTop
          // 滚动高度+浏览器可视区域高度 比 偏移高度加元素高度 少，下面溢出
        const bottomOverflow = window.scrollY+document.documentElement.clientHeight < offsetTop+offsetHeight
        // 滚动宽度 超过 偏移宽度，左侧溢出
        const leftOverflow = window.scrollX > offsetLeft;
          // 滚动宽度 + 浏览器可视区域宽度 小于 偏移宽度 + 元素宽度， 右侧溢出
        const rightOverflow = window.scrollX+document.documentElement.clientWidth < offsetLeft + offsetWidth

        const removeClassAll = () => {
          dom.classList.remove('relative', 'sticky-bottom', 'sticky-top', 'sticky-left', 'sticky-right');
        }

        console.log(topOverflow, bottomOverflow, leftOverflow, rightOverflow);
        removeClassAll();
        if (topOverflow) {
          dom.classList.add('sticky-top');
        }
        if (bottomOverflow) {          
          dom.classList.add('sticky-bottom');

        }
        if (leftOverflow) {
          dom.classList.add('sticky-left');
        }
        if (rightOverflow) {
          dom.classList.add('sticky-right');
        }
        
        if (!(topOverflow || bottomOverflow || leftOverflow || rightOverflow)) {
          dom.classList.add('relative');
        }
      }

      const isOverflowDeBounce = debounce(isOverflow, 16);

      // 事件监听
      window.addEventListener('scroll', () => {
        isOverflowDeBounce();
      });
    }

```

# 注意
* 脱离文档流的好处一定时间上可以提升浏览器的回流重绘


# 来源

https://developer.mozilla.org/zh-CN/docs/Web/CSS/position

