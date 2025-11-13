---
title: LeetCode刷题（旋转链表、分隔链表）
published: 2025-11-09
updated: 2025-11-09
description: '旋转链表、分隔链表'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 旋转链表

## 题目描述

给你一个链表的头节点 `head` ，旋转链表，将链表每个节点向右移动 `k` 个位置。

 

**示例 1：**

![345](../images/345.jpg)

```
输入：head = [1,2,3,4,5], k = 2
输出：[4,5,1,2,3]
```

**示例 2：**

![346](../images/346.jpg)

```
输入：head = [0,1,2], k = 4
输出：[2,0,1]
```



## 题解

对于旋转而言旋转次数可以简化为旋转 k % length次，我们计算出链表的长度num，在计算需要旋转的次数k，设置一个p指针指向链表最后一个元素让其指向头部节点，组成一个环

对于向右移动而言需要移动num - k次再断开链表，对于向左移动则需要移动k再断开链表，返回断开后的首节点即可

```java
class Solution {
    public ListNode rotateRight(ListNode head, int k) {
        if(head == null) return null;
    
        ListNode p = new ListNode();
        p.next = head;

        int num = 0;
        while(p.next != null) {
            num++;
            p = p.next;
        }
        k = k % num;
        p.next = head;
        for(int i = 0; i < num - k; i++) {
            p = p.next;
        }
        ListNode result = p.next;
        p.next = null;
        return result;
    }
}
```



# 分割链表

## 题目描述

给你一个链表的头节点 `head` 和一个特定值 `x` ，请你对链表进行分隔，使得所有 **小于** `x` 的节点都出现在 **大于或等于** `x` 的节点之前。

你应当 **保留** 两个分区中每个节点的初始相对位置。

 

**示例 1：**

![347](../images/347.jpg)

```
输入：head = [1,4,3,2,5,2], x = 3
输出：[1,2,2,4,3,5]
```

**示例 2：**

```
输入：head = [2,1], x = 2
输出：[1,2]
```



## 题解

设立两个链表，分别用尾插法存放两个不同范围的节点，最后将其合并，这里注意要将第二部部分末尾置为空，不然就会有环的情况

```java
class Solution {
    public ListNode partition(ListNode head, int x) {
        ListNode first = new ListNode();
        ListNode p1 = first;
        ListNode second = new ListNode();
        ListNode p2 = second;
        while(head != null) {
            if(head.val < x) {
                p1.next = head;
                p1 = p1.next;
            } else {
                p2.next = head;
                p2 = p2.next;
            }
            head = head.next;
        } 
        p2.next = null;
        p1.next = second.next;
        return first.next;
    }
}
```

