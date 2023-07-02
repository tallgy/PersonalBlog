---
title: 删除链表的倒数第 N 个结点

date: 2023-07-02 17:07:21
tags:
 - 算法
 - 双指针
categories:
 - 算法
 - 中等
 - 双指针
---

# 删除链表的倒数第 N 个结点

```
https://leetcode.cn/problems/remove-nth-node-from-end-of-list/description/
```

## 描述

给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

 

示例 1：


输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
示例 2：

输入：head = [1], n = 1
输出：[]
示例 3：

输入：head = [1,2], n = 1
输出：[1]
 

提示：

链表中结点的数目为 sz
1 <= sz <= 30
0 <= Node.val <= 100
1 <= n <= sz


## 算法 双指针 快慢指针

思路很简单，就是使用双指针，一个指针先往后面走n次，然后慢指针再开始走。
然后快指针走到了结束之后，慢指针的位置就处于是被删除节点的位置。
然后就将被删除的节点进行操作即可。
需要考虑的边界条件：如果被删除的节点是第一个，那么需要考虑。


```


/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} n
 * @return {ListNode}
 */
var removeNthFromEnd = function (head, n) {
  let leftNode = head, rightNode = head;
  let prantNode = head;

  while (rightNode) {
    // 快指针的移动
    rightNode = rightNode.next;

    if (n <= 0) {
      // 因为快指针移动到末尾时，慢指针的位置在被删除的位置上面。所以需要记录前一个指针
      prantNode = leftNode;
      // 慢指针的移动
      leftNode = leftNode.next;
    }
    n--;
  }

  // 如果慢指针没有移动，那么就直接 head next
  if (leftNode === head) {
    return head.next;
  }

  // 删除指针
  prantNode.next = leftNode.next;
  return head;
};


```


