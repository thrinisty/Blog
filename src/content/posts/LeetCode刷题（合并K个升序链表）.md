---
title: LeetCode刷题（合并K个升序链表）
published: 2025-06-10
updated: 2025-06-10
description: '合并K个升序链表'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 合并K个升序链表

## 题目描述

给你一个链表数组，每个链表都已经按升序排列。

请你将所有链表合并到一个升序链表中，返回合并后的链表。

 

**示例 1：**

```
输入：lists = [[1,4,5],[1,3,4],[2,6]]
输出：[1,1,2,3,4,4,5,6]
解释：链表数组如下：
[
  1->4->5,
  1->3->4,
  2->6
]
将它们合并到一个有序链表中得到。
1->1->2->3->4->4->5->6
```

**示例 2：**

```
输入：lists = []
输出：[]
```

**示例 3：**

```
输入：lists = [[]]
输出：[]
```



## 题解

解法一：

之前做过一道合并两个链表的题目，尝试通过逐个取出集合中的链表进行合并

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        if(lists.length == 0 || lists == null) {
            return null;
        }
        ListNode result = null;
        for(ListNode list : lists) {
            result = mergeList(result, list);
        }
        return result;
    }

    public ListNode mergeList(ListNode listA, ListNode listB) {
        ListNode result = new ListNode();
        ListNode p = result;
        while(listA != null && listB != null) {
            if(listA.val <= listB.val) {
                p.next = listA;
                listA = listA.next;
                p = p.next;
            } else {
                p.next = listB;
                listB = listB.next;
                p = p.next;
            }
        }
        p.next = listA == null ? listB : listA;
        return result.next;
    }
}
```

原本想着应该不可以通过测试用例之类的，但是竟然过了，只是执行速度非常缓慢，时间复杂度是O(N*k^2)，其中k是链表数量，N是平均链表长度



解法二：归并排序，合并两个链表的方式和第一种一致

可以用归并排序，递归地将链表两两合并，相比于第一种方式而言不用重复的合并result链表，节省了很多的时间，这一题用数组的方式给出链表应该也是鼓励使用这种方式

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        if(lists.length == 0 || lists == null) return null;
        return merge(lists, 0, lists.length - 1);
    }

    public ListNode merge(ListNode[] lists, int left, int right) {
        if(left == right) {
            return lists[left];
        }
        int mid = left + (right - left) / 2;
        ListNode leftList = merge(lists, left, mid);
        ListNode rightList = merge(lists, mid + 1, right);
        return mergeList(leftList, rightList);
    }

    public ListNode mergeList(ListNode listA, ListNode listB) {
        ListNode result = new ListNode();
        ListNode p = result;
        while(listA != null && listB != null) {
            if(listA.val <= listB.val) {
                p.next = listA;
                listA = listA.next;
                p = p.next;
            } else {
                p.next = listB;
                listB = listB.next;
                p = p.next;
            }
        }
        p.next = listA == null ? listB : listA;
        return result.next;
    }
}
```

虽然说这一题是困难，但是因为之前学习数据结构和算法的时候对于链表和分治的思想掌握比较牢靠，其实也不难理解，这道题是LeetCode100的第50道了，刷的还蛮快的嘛都一半了，继续加油！！
