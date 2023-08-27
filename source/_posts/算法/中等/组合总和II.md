---
title: 组合总和II
tags:
  - 算法
  - 中等
categories:
  - 算法
  - 中等
date: 2023-08-27 23:12:06
---

# 组合总和 II


```
https://leetcode.cn/problems/combination-sum-ii/
```

## 描述

给定一个候选人编号的集合 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的每个数字在每个组合中只能使用 一次 。

注意：解集不能包含重复的组合。 

 

示例 1:

输入: candidates = [10,1,2,7,6,1,5], target = 8,
输出:
[
[1,1,6],
[1,2,5],
[1,7],
[2,6]
]
示例 2:

输入: candidates = [2,5,2,1,2], target = 5,
输出:
[
[1,2,2],
[5]
]
 

提示:

1 <= candidates.length <= 100
1 <= candidates[i] <= 50
1 <= target <= 30

## 算法

大致逻辑和 数组总和差不多
只是说多了一个部分就是对于数组有一个简单的去重处理
加上每个元素只能用一次

```JavaScript

/**
 * @param {number[]} candidates
 * @param {number} target
 * @return {number[][]}
 */
var combinationSum2 = function(candidates, target) {
  const res = [];
  const len = candidates.length;
  // 先排序
  candidates.sort((a, b) => a-b);

  const recur = (value, curArr, index) => {
    if (value > target) {
      return;
    }
    if (value === target) {
      res.push(curArr);
      return;
    }

    for (let i=index; i<len; i++) {
      // 用这个，解决重复的情况
      if (i>index && candidates[i] === candidates[i-1]) {
        continue;
      }
      // 这里从 i变成了 i+1 所以，解决了每个元素只能用一次
      recur(value+candidates[i], [...curArr, candidates[i]], i+1);
    }
  }

  recur(0, [], 0);

  return res;
};

```


