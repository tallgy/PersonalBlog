---
title: x的平方根
date: 2023-07-08 15:49:51
tags:
 - 算法
categories:
 - 算法
 - 简单
 - 二分法
---

#  x 的平方根

```
https://leetcode.cn/problems/sqrtx/description/
```

## 描述

给你一个非负整数 x ，计算并返回 x 的 算术平方根 。

由于返回类型是整数，结果只保留 整数部分 ，小数部分将被 舍去 。

注意：不允许使用任何内置指数函数和算符，例如 pow(x, 0.5) 或者 x ** 0.5 。

 

示例 1：

输入：x = 4
输出：2
示例 2：

输入：x = 8
输出：2
解释：8 的算术平方根是 2.82842..., 由于返回类型是整数，小数部分将被舍去。
 

提示：

0 <= x <= 231 - 1


## 算法


### 迭代法

就很简单，从1开始迭代，
迭代到第一个大于她的数 x
return x-1



### 二分法

使用二分法进行计算，找到相同的，或者找到无限逼近的数


```JavaScript

/**
 * @param {number} x
 * @return {number}
 */
var mySqrt = function(x) {
  let left = 0, right = x;
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    const midSquare = mid * mid;
    if (midSquare > x) {
      right = mid - 1;
    } else if (midSquare < x) {
      left = mid + 1;
    } else {
      return mid;
    }
  }

  // 这里可以用 right 或者 left-1 因为首先如果出来了，说明是没有找到
  // 那么首先，此时 left 肯定是大于了 right
  // 其次，两个值是属于无限逼近的情况，所以 left 仅仅比 right 大 1
  // left - 1
  return right;
};

```


### 牛顿迭代法

有点高深，我直接放弃。
大致意思就是通过求 斜率，然后通过斜率的垂直线相交点继续求斜率，无限往下。
这个转移方程的驱动速率比二分法更快。

