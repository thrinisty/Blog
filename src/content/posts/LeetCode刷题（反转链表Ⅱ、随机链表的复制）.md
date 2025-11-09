---
title: LeetCode刷题（反转链表Ⅱ、随机链表的复制）
published: 2025-11-07
updated: 2025-11-07
description: '反转链表Ⅱ、随机链表的复制'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 反转链表Ⅱ

## 题目描述

给你单链表的头指针 `head` 和两个整数 `left` 和 `right` ，其中 `left <= right` 。请你反转从位置 `left` 到位置 `right` 的链表节点，返回 **反转后的链表** 。

 

**示例 1：**

![341](../images/341.jpg)

```
输入：head = [1,2,3,4,5], left = 2, right = 4
输出：[1,4,3,2,5]
```

**示例 2：**

```
输入：head = [5], left = 1, right = 1
输出：[5]
```



## 题解

先遍历left-1，找到要交换链表起始节点的上一个节点lastNode，用cur指针指向交换的起始节点

每次将元素提前算法：用next指向要放入点前面的节点，cur指向交换结点的下一个节点等待下次交换，将next节点放入lastNode节点的下一位，依次交换right-left次即可

```java
class Solution {
    public ListNode reverseBetween(ListNode head, int left, int right) {
        ListNode result = new ListNode();
        result.next = head;
        ListNode lastNode = result;
        for(int i = 1; i < left; i++) {
            lastNode = lastNode.next;
        }
        ListNode cur = lastNode.next;
        ListNode next;
        for(int i = 0; i < right - left; i++) {
            next = cur.next;
            cur.next = next.next;
            next.next = lastNode.next;
            lastNode.next = next;
        }
        return result.next;
    }
}
```



# 随机链表的复制

## 题目描述

你一个长度为 `n` 的链表，每个节点包含一个额外增加的随机指针 `random` ，该指针可以指向链表中的任何节点或空节点。

构造这个链表的 **[深拷贝](https://baike.baidu.com/item/深拷贝/22785317?fr=aladdin)**。 深拷贝应该正好由 `n` 个 **全新** 节点组成，其中每个新节点的值都设为其对应的原节点的值。新节点的 `next` 指针和 `random` 指针也都应指向复制链表中的新节点，并使原链表和复制链表中的这些指针能够表示相同的链表状态。**复制链表中的指针都不应指向原链表中的节点** 。

例如，如果原链表中有 `X` 和 `Y` 两个节点，其中 `X.random --> Y` 。那么在复制链表中对应的两个节点 `x` 和 `y` ，同样有 `x.random --> y` 。

返回复制链表的头节点。

用一个由 `n` 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 `[val, random_index]` 表示：

- `val`：一个表示 `Node.val` 的整数。
- `random_index`：随机指针指向的节点索引（范围从 `0` 到 `n-1`）；如果不指向任何节点，则为 `null` 。

你的代码 **只** 接受原链表的头节点 `head` 作为传入参数。



## 题解

合并+分离，分三次循环，第一次循环创建后置节点，第二次跨格将指针指向对应的后置节点，第三次将合并的链表拆分

```java
class Solution {
    public Node copyRandomList(Node head) {
        if(head == null) {
            return null;
        }
        Node p = head;
        while(p != null) {
            Node node = new Node(p.val);
            node.next = p.next;
            p.next = node;
            p = node.next;
        }
        p = head;
        while(p != null) {
            if(p.random != null) {
                p.next.random = p.random.next;
            }
            p = p.next.next;
        }
        Node result = new Node(-1);
        p = head;
        Node cur = result;
        while(p != null) {
            cur.next = p.next;
            cur = cur.next;
            p.next = p.next.next;
            p = p.next;
        }
        return result.next;
    }
}
```

