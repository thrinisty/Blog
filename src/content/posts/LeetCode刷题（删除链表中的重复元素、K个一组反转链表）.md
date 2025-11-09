---
title: LeetCode刷题（删除链表中的重复元素、K个一组反转链表）
published: 2025-11-08
updated: 2025-11-08
description: '删除链表中的重复元素、K个一组反转链表'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 删除链表中的重复元素

## 题目描述

给定一个已排序的链表的头 `head` ， *删除所有重复的元素，使每个元素只出现一次* 。返回 *已排序的链表* 。

**示例 1：**

```
输入：head = [1,1,2]
输出：[1,2]
```

**示例 2：**

```
输入：head = [1,1,2,3,3]
输出：[1,2,3]
```



## 题解

当p的下一个节点不为空的时候比较当前节点和下一节点的数值，如果相同则将当前指针指向下一个的下一个节点（移除当前节点的后一节点），如果不同则将当前指针指向下一节点等待下一次判断

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head == null) return null;
        ListNode p = head;
        while(p.next != null) {
            if(p.val == p.next.val) {
                p.next = p.next.next;
            } else {
                p = p.next;
            }
        }
        return head;
    }
}
```



# 删除链表中的重复元素Ⅱ

## 题目描述

定一个已排序的链表的头 `head` ， *删除原始链表中所有重复数字的节点，只留下不同的数字* 。返回 *已排序的链表* 。

**示例 1：**

```
输入：head = [1,2,3,3,4,4,5]
输出：[1,2,5]
```

**示例 2：**

```
输入：head = [1,1,1,2,3]
输出：[2,3]
```

 

## 题解

这一道题是上一道题的拓展，将重复的元素给完全删除不保留其单一节点

思路有些改变，因为需要删除重复的所有节点包括起始的节点，我们需要一个记录节点的前一节点，选取该节点后的第一个数作为基准，依次删除值相同的节点

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head == null) return null;
        ListNode result = new ListNode();
        result.next = head;
        ListNode p = result;
        while(p.next != null && p.next.next != null) {
            if(p.next.val == p.next.next.val) {
                int value = p.next.val;
                while(p.next != null && p.next.val == value) {
                    p.next = p.next.next;
                }
            } else {
                p = p.next;
            }
        }
        return result.next;
    }
}
```



# K个一组反转链表

## 题目描述

给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

 

**示例 1：**

![343](../images/343.jpg)

```
输入：head = [1,2,3,4,5], k = 2
输出：[2,1,4,3,5]
```

**示例 2：**

![344](../images/344.jpg)

```
输入：head = [1,2,3,4,5], k = 3
输出：[3,2,1,4,5]
```

 

**提示：**

- 链表中的节点数目为 `n`
- `1 <= k <= n <= 5000`
- `0 <= Node.val <= 1000`

 

## 题解

这是我写的方法：一次运用之前的部分反转的思路进行反转每个k区间，直到next为空的时候停止，这种方式如果最后的区间不满足k个也会进行反转

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode result = new ListNode();
        result.next = head;
        ListNode last = result;
        ListNode cur = last.next;
        while(true) {
            if(cur == null) break;
            for(int i = 1; i < k; i++) {
                ListNode next = cur.next;
                if(next == null) break;
                cur.next = next.next;
                next.next = last.next;
                last.next = next;
            }
            last = cur;
            cur = last.next;
        }
        return result.next;
    }
}
```

应该在进行数量的判断，如果不满足K个数则提前返回

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode result = new ListNode();
        result.next = head;
        ListNode last = result;
        ListNode cur = last.next;
        while(true) {
            if(cur == null) break;
            int count = 0;
            ListNode p = cur;
            for(int i = 0; i < k; i++) {
                if(p == null) {
                    break;
                }
                count++;
                p = p.next;
            }
            if(count != k) break;
            for(int i = 1; i < k; i++) {
                ListNode next = cur.next;
                cur.next = next.next;
                next.next = last.next;
                last.next = next;
            }
            last = cur;
            cur = last.next;
        }
        return result.next;
    }
}
```

