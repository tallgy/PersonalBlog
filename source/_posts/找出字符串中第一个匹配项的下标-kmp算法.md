---
title: 找出字符串中第一个匹配项的下标-kmp算法
tags:
  - 算法
  - 简单
categories:
  - 算法
  - 简单
date: 2023-08-20 21:33:02
---

# 找出字符串中第一个匹配项的下标

```
https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string
```

## 描述


给你两个字符串 haystack 和 needle ，请你在 haystack 字符串中找出 needle 字符串的第一个匹配项的下标（下标从 0 开始）。如果 needle 不是 haystack 的一部分，则返回  -1 。

 

示例 1：

输入：haystack = "sadbutsad", needle = "sad"
输出：0
解释："sad" 在下标 0 和 6 处匹配。
第一个匹配项的下标是 0 ，所以返回 0 。
示例 2：

输入：haystack = "leetcode", needle = "leeto"
输出：-1
解释："leeto" 没有在 "leetcode" 中出现，所以返回 -1 。
 

提示：

1 <= haystack.length, needle.length <= 104
haystack 和 needle 仅由小写英文字符组成


## 算法


### 暴力解法

暴力解法比较简单
就是简单的两层循环即可

```JavaScript

var strStr = function(haystack, needle) {
    const n = haystack.length, m = needle.length;
    for (let i = 0; i + m <= n; i++) {
        let flag = true;
        for (let j = 0; j < m; j++) {
            if (haystack[i + j] != needle[j]) {
                flag = false;
                break;
            }
        }
        if (flag) {
            return i;
        }
    }
    return -1;
};

```


### KMP 算法


对于这个存在一个 KMP 算法
比如，在一些字符串来说，就算当前匹配失败了，也不需要直接从0开始
可以从中间开始，减少了很多匹配的次数。

```JavaScript



```
