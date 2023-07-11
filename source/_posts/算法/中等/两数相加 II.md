---
title: 两数相加 II
date: 2023-07-11 14:41:08
tags:
 - 算法
categories:
 - 算法
 - 中等
---

# 两数相加 II

```
https://leetcode.cn/problems/add-two-numbers-ii/description/
```

## 描述


给你两个 非空 链表来代表两个非负整数。数字最高位位于链表开始位置。它们的每个节点只存储一位数字。将这两数相加会返回一个新的链表。

你可以假设除了数字 0 之外，这两个数字都不会以零开头。

 

示例1：



输入：l1 = [7,2,4,3], l2 = [5,6,4]
输出：[7,8,0,7]
示例2：

输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[8,0,7]
示例3：

输入：l1 = [0], l2 = [0]
输出：[0]
 

提示：

链表的长度范围为 [1, 100]
0 <= node.val <= 9
输入数据保证链表代表的数字无前导 0



## 算法


### 使用堆栈

先将链表的数据放入堆栈中，然后再 pop 出来进行相加，
注意就是最后一个地方，还会有一个进位需要考虑


```

/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = function(l1, l2) {
  const stack1 = [], stack2 = [];

  while (l1) {
    stack1.push(l1.val);
    l1 = l1.next;
  }

  while (l2) {
    stack2.push(l2.val);
    l2 = l2.next;
  }

  let preNode = null;
  let carry = 0;

  while (stack1.length || stack2.length || carry) {
    const value = (stack1.pop() || 0) + (stack2.pop() || 0) + carry;
    const node = new ListNode(value % 10);
    carry = Math.floor(value / 10);
    node.next = preNode;
    preNode = node;
  }

  return preNode;
};

```


