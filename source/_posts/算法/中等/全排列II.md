---
title: 全排列II
tags:
  - 算法
  - 中等
categories:
  - 算法
  - 中等
date: 2023-09-03 14:40:12
---

# 全排列 II

```
https://leetcode.cn/problems/permutations-ii
```

## 描述

给定一个可包含重复数字的序列 nums ，按任意顺序 返回所有不重复的全排列。

 

示例 1：

输入：nums = [1,1,2]
输出：
[[1,1,2],
 [1,2,1],
 [2,1,1]]
示例 2：

输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
 

提示：

1 <= nums.length <= 8
-10 <= nums[i] <= 10

## 算法

### 回溯 同全排列相同的解法

解法和全排列相同，只是在 push 结果之前，加了一个判断是否是已经存在的数据

```JavaScript

/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var permuteUnique = function(nums) {
  const res = [];
  const len = nums.length;
  const arrObj = {};

  const recur = (arr, index) => {
    if (index === len-1 && !arrObj[arr.toString()]) {
      res.push([...arr]);
      arrObj[arr.toString()] = true;
      return;
    }

    for (let i=index; i<len; i++) {
      [arr[i], arr[index]] = [arr[index], arr[i]];
      recur(arr, index+1);
      [arr[i], arr[index]] = [arr[index], arr[i]];
    }
  }

  recur(nums, 0);

  return res;
};

```


### 搜索回溯

首先是排序，将数组按顺序排列，
保证了，相同的数字是挨着的，然后如果两个数字相同，
但是前面的数字已经没有被使用，说明了这个所产生的顺序一定是已经回存在的，忽略

比如： 1、1、2 先排序，保证了顺序相同问题。
如果 flag[0] = true 代表了 index 0 已经被使用了
那么第一次的 1 
那么：0和1的值相等的情况下， index[0] = true 代表了前面是被使用了，说明了这个顺序是第一次，所以可以使用

第二次的 1
 flag[0] = false， 因为 0、1 两个值相等， 0 为false ，说明了 前面那个没有被使用，但是按照顺序来说，前面那个肯定是被使用过的
  所以此时是第二个，那么就可以直接跳过。


```JavaScript
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var permuteUnique = function(nums) {
  const res = [];
  const len = nums.length;
  nums.sort((a, b) => a-b);
  const flags = new Array(len).fill(false);

  const recur = (arr, index) => {
    if (arr.length === len) {
      res.push([...arr]);
      return;
    }

    for (let i=0; i<len; i++) {
      const v = nums[i];
      // 如果使用了，那么就跳过
      // 如果 前面的数字和当前相同，且前面没有被使用，就说明了，这个所产生的顺序一定是会重复的。
      if (flags[i] || (i>0 && v === nums[i-1] && !flags[i-1])) {
        continue;
      }

      flags[i] = true;
      recur([...arr, v])
      flags[i] = false;

    }
  }

  recur([]);

  return res;
};

```
