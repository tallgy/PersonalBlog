---
title: css-hover和focus顺序
date: 2022-02-04 17:16:47
tags:
 - 随笔
 - CSS
categories:
 - CSS
 - 随笔
---



#  hover和focus的顺序问题

```
div:hover { }
div:focus { }
```

​		最近在写项目的时候无意间发现的问题，对于hover和触发和focus的触发问题。这个其实在学的时候就已经知道了，但是因为用的较少，所以还忘记了这个问题。

​		简单来说就是 hover需要放在focus的前面，因为hover的触发比focus触发的时机问题。会先focus然后再hover(应该)，但是在css里面，对于优先级相同的。在后面的会覆盖前面的。

​		通过这个我们也要知道关于冒泡和捕获触发的问题也要注意这个情况。事件是先捕获，然后再冒泡。

