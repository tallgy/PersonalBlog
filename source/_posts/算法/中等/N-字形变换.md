---
title: N-字形变换
tags:
  - 算法
  - 中等
categories:
  - 算法
  - 中等
date: 2023-08-18 19:33:11
---

# N-字形变换

```
https://leetcode.cn/problems/zigzag-conversion/description/
```

## 描述

将一个给定字符串 s 根据给定的行数 numRows ，以从上往下、从左到右进行 Z 字形排列。

比如输入字符串为 "PAYPALISHIRING" 行数为 3 时，排列如下：

P   A   H   N
A P L S I I G
Y   I   R
之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如："PAHNAPLSIIGYIR"。

请你实现这个将字符串进行指定行数变换的函数：

string convert(string s, int numRows);
 

示例 1：

输入：s = "PAYPALISHIRING", numRows = 3
输出："PAHNAPLSIIGYIR"
示例 2：
输入：s = "PAYPALISHIRING", numRows = 4
输出："PINALSIGYAHRPI"
解释：
P     I    N
A   L S  I G
Y A   H R
P     I
示例 3：

输入：s = "A", numRows = 1
输出："A"
 

提示：

1 <= s.length <= 1000
s 由英文字母（小写和大写）、',' 和 '.' 组成
1 <= numRows <= 1000


## 算法

### 按照正常思路

就是创建一个 numRows 的数组，
然后 依次 push 就可以放进去了
用了一个 getIndex 方法来获取正常的 index
最后再使用 reduce 来返回字符串。

```JavaScript

/**
 * @param {string} s
 * @param {number} numRows
 * @return {string}
 */
var convert = function(s, numRows) {
  const len = s.length;
  const res = new Array(numRows).fill(0).map(_ => []);
  let flag = false;

  const getIndex = (i, j) => {
    if (numRows === 1) {
      return [0, j+1]
    }
    if (i === 0) {
      flag = false;
    }
    if (i === numRows-1 || flag) {
      flag = true;
      return [i-1, j+1];
    }
    
    return [i+1, j]
  }

  let i = 0, j = 0;
  for (let z=0; z<len; z++) {
    res[i].push(s[z]);
    [i, j] = getIndex(i, j);
  }

  return res.reduce((pre, cur) => pre + cur.join(''), '');
};

```


### 直接构造

```

其实通过计算发现规律
n=4

0     6
1   5 7
2 4   8
3     9


规律：
t = 2*n-2 = 6
0     t
1   5 t+1
2 4   t+2
3     t+3

重点在于 中间 4、5 的规律，但是我们可以发现
5到t+1的距离其实就是，他们之间差了一个 i 的高度。
所以得出 5 = t+1-2*i

```

```JavaScript

/**
 * @param {string} s
 * @param {number} numRows
 * @return {string}
 */
var convert = function(s, numRows) {
  const len = s.length;

  // 特殊情况。
  if (numRows === 1 || numRows >= len) {
    return s;
  }

  const t = 2 * numRows - 2;
  const res = [];

  for (let i=0; i<numRows; i++) {
    for (let j=i; j<len; j+=t) {
      res.push(s[j]);
      // 判断是否应该二次push，只要不是首尾就应该push
      if (0<i && i<numRows-1) {
        res.push(s[j+t-2*i]); // 当前周期的第二个字符
      }
    }
  }

  return res.join('');
};

```
