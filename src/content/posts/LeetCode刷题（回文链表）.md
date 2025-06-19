---
title: LeetCode刷题（回文链表）
published: 2025-06-19
updated: 2025-06-19
description: '回文链表，运用中间节点和链表反转求解'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 回文链表

## 题目描述

给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false` 。

 

**示例 1：**

![191](../images/191.jpg)

```
输入：head = [1,2,2,1]
输出：true
```

**示例 2：**

![192](../images/192.jpg)

```
输入：head = [1,2]
输出：false
```

## 题解

解法一：运用List集合辅助判断，思路类似于判断数组是否是回文

```java
class Solution {
    public boolean isPalindrome(ListNode head) {
        List<Integer> list = new ArrayList<>();
        ListNode p = head;
        while(p != null) {
            list.add(p.val);
            p = p.next;
        }
        int n = list.size();
        for(int i = 0, j = n - 1; i <= j; i++, j--) {
            if(list.get(i) != list.get(j)) {
                return false;
            }
        }
        return true;
    }
}
```

但是题目进阶要求空间复杂度为O(1)，不可以使用额外的存储空间



解法二：运用中间节点和反转链表求解

先来回顾一下运用头插法实现链表反转的方法，我们在后续中会使用到这个函数

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode result = new ListNode();
        ListNode p = head;
        ListNode temp;
        while(p != null) {
            temp = p.next;
            p.next = result.next;
            result.next = p;
            p = temp;
        }
        return result.next;
    }
}
```

我们再普及一个算法，用于输入一个链表的头节点输出他的中间节点，内部由快慢指针实现

```java
public ListNode returnHalf(ListNode head) {
        ListNode fast = head;
        ListNode slow = head;
        while (fast.next != null && fast.next.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        return slow;
    }
```



运用以上的两个函数，我们只需要将中间节点部分后的节点进行反转，在头节点和中间节点之后按照链表顺序依次判断节点内容是否相等即可

```java
class Solution {
    public boolean isPalindrome(ListNode head) {
        if(head == null) {
            return true;
        }
        ListNode p1 = halfNode(head);
        ListNode p2 = reverseList(p1.next);
        p1 = head;
        while(p2 != null) {
            if(p1.val != p2.val) {
                return false;
            }
            p1 = p1.next;
            p2 = p2.next;
        }
        return true;
    }

    public ListNode reverseList(ListNode head) {
        ListNode result = new ListNode();
        ListNode p = head;
        ListNode temp;
        while(p != null) {
            temp = p.next;
            p.next = result.next;
            result.next = p;
            p = temp;
        }
        return result.next;
    }

    public ListNode halfNode(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while(fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }
}
```

今天的练习属于是老题新解，拓展了以下思路，运用先前的链表反转和取中间节点来解决链表回文的判断
