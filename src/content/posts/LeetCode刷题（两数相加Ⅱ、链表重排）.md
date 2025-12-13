---
title: LeetCode刷题（两数相加Ⅱ、链表重排）
published: 2025-12-06
updated: 2025-12-06
description: '两数相加Ⅱ、链表重排'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 两数相加Ⅱ

## 题目描述

给定两个 **非空链表** `l1`和 `l2` 来代表两个非负整数。数字最高位位于链表开始位置。它们的每个节点只存储一位数字。将这两数相加会返回一个新的链表。

可以假设除了数字 0 之外，这两个数字都不会以零开头。

**示例 1：**

![495](../images/495.png)

```
输入：l1 = [7,2,4,3], l2 = [5,6,4]
输出：[7,8,0,7]
```

**示例 2：**

```
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[8,0,7]
```

**示例 3：**

```
输入：l1 = [0], l2 = [0]
输出：[0]
```

 

## 题解

解法一：通过反转链表求解，除此之外思路和两数相加Ⅰ一致

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        l1 = reverseList(l1);
        l2 = reverseList(l2);
        int count = 0;
        ListNode result = new ListNode();
        ListNode p = result;
        while (l1 != null || l2 != null || count != 0) {
            int num1 = l1 == null ? 0 : l1.val;
            int num2 = l2 == null ? 0 : l2.val;
            int sum = num1 + num2 + count;
            count = sum / 10;
            p.next = new ListNode(sum % 10);
            p = p.next;
            if(l1 != null) l1 = l1.next;
            if(l2 != null) l2 = l2.next;
        }
        return reverseList(result.next);
    }

    public ListNode reverseList(ListNode head) {
        ListNode result = new ListNode();
        while (head != null) {
            ListNode temp = head.next;
            head.next = result.next;
            result.next = head;
            head = temp;
        }
        return result.next;
    }
}
```

解法二：通过栈进行求解，通过弹栈可以有效地取出低位的数字，再根据头插法进行结果构造

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        Stack<Integer> s1 = new Stack<>();
        Stack<Integer> s2 = new Stack<>();
        while(l1 != null) {
            s1.push(l1.val);
            l1 = l1.next;
        }
        while(l2 != null) {
            s2.push(l2.val);
            l2 = l2.next;
        }
        int count = 0;
        ListNode result = new ListNode();
        while(!s1.isEmpty() || !s2.isEmpty() || count != 0) {
            int num1 = s1.isEmpty() ? 0 : s1.pop();
            int num2 = s2.isEmpty() ? 0 : s2.pop();
            int sum = num1 + num2 + count;
            count = sum / 10;
            ListNode node = new ListNode(sum % 10);
            node.next = result.next;
            result.next = node;
        }
        return result.next;
    }
}
```



# 链表重排

## 题目描述

给定一个单链表 `L` 的头节点 `head` ，单链表 `L` 表示为：

` L0 → L1 → … → Ln-1 → Ln `
请将其重新排列后变为：

```
L0 → Ln → L1 → Ln-1 → L2 → Ln-2 → …
```

不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

 

**示例 1：**

![496](../images/496.png)

```
输入: head = [1,2,3,4]
输出: [1,4,2,3]
```

**示例 2：**

![497](../images/497.png)

```
输入: head = [1,2,3,4,5]
输出: [1,5,2,4,3]
```



## 题解

构造辅助方法反转链表和取出中间节点，将后半段链表反转后与前段进行合并

```java
class Solution {
    public void reorderList(ListNode head) {
        ListNode temp = halfNode(head);
        ListNode halfList = temp.next;
        temp.next = null;
        halfList = reverse(halfList);
        ListNode result = new ListNode();
        ListNode p = result;
        while(halfList != null) {
            p.next = head;
            head = head.next;
            p = p.next;

            p.next = halfList;
            halfList = halfList.next;
            p = p.next;
        }
        if(head != null) {
            p.next = head;
            head = head.next;
            p = p.next;
        }
        head = result.next;
    }

    public ListNode halfNode(ListNode head) {
        ListNode fast = head;
        ListNode slow = head;
        while(fast.next != null && fast.next.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        return slow;
    }

    public ListNode reverse(ListNode head) {
        ListNode result = new ListNode();
        ListNode temp;
        while(head != null) {
            temp = head.next;
            head.next = result.next;
            result.next = head;
            head = temp;
        }
        return result.next;
    }
}
```

