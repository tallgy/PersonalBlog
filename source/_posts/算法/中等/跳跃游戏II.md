---
title: 跳跃游戏II
date: 2023-07-19 15:38:06
tags:
 - 算法
categories:
 - 算法
 - 中等
---

# 跳跃游戏 II

```
https://leetcode.cn/problems/jump-game-ii/description/
```


## 描述

给定一个长度为 n 的 0 索引整数数组 nums。初始位置为 nums[0]。

每个元素 nums[i] 表示从索引 i 向前跳转的最大长度。换句话说，如果你在 nums[i] 处，你可以跳转到任意 nums[i + j] 处:

0 <= j <= nums[i] 
i + j < n
返回到达 nums[n - 1] 的最小跳跃次数。生成的测试用例可以到达 nums[n - 1]。

 

示例 1:

输入: nums = [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
示例 2:

输入: nums = [2,3,0,1,4]
输出: 2
 

提示:

1 <= nums.length <= 104
0 <= nums[i] <= 1000
题目保证可以到达 nums[n-1]



## 算法

### 逆序统计最小值

从末尾指针开始，然后每次逆序统计，当前指针到最后指针需要几步，然后将最小值记录下来并返回


```JavaScript

/**
 * @param {number[]} nums
 * @return {number}
 */
var jump = function(nums) {
  const len = nums.length;
  const dp = new Array(len).fill(0);

  for (let i=len-2; i>=0; i--) {
    if (len-i-1 <= nums[i]) {
      dp[i] = 1;
    } else {
      dp[i] = Infinity;
      for (let j=i+1; j<=nums[i]+i; j++) {
        dp[i] = Math.min(dp[i], dp[j]+1)
      }
    }
  }
  return dp[0];
};

```


### 正序贪心算法

正序，从开始位置，统计到结束位置，然后通过贪心记录每次可以跳转的最长位置，没超过一次最长位置，代表了步数需要加一。
注意：不会到真正的最后一步，因为如果在最后一步，就不需要跳了，所以要 -1 ，否则可能会出现多一次的情况

```JavaScript

/**
 * @param {number[]} nums
 * @return {number}
 */
var jump = function(nums) {
  let maxPosition = 0;
  let end = 0;
  let step = 0;
  const len = nums.length;
  for (let i=0; i<len-1; i++) {
    maxPosition = Math.max(maxPosition, nums[i]+i);
    if (i === end) {
      end = maxPosition;
      step++;
    }
  }
  return step
};


```
