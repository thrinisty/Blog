---
title: LeetCode刷题（两数相加、搜索二维矩阵）
published: 2025-09-29
updated: 2025-09-29
description: '链表两数相加、搜索二维矩阵'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 两数相加

## 题目描述

给你两个 **非空** 的链表，表示两个非负的整数。它们每位数字都是按照 **逆序** 的方式存储的，并且每个节点只能存储 **一位** 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

 

**示例 1：**

![222](../images/222.jpg)

```
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[7,0,8]
解释：342 + 465 = 807.
```

**示例 2：**

```
输入：l1 = [0], l2 = [0]
输出：[0]
```

**示例 3：**

```
输入：l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
输出：[8,9,9,9,0,0,0,1]
```



## 题解

解法一：先通过直接将两个链表进行合并，不进行进位的处理，再对合并的链表作进位处理

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode result = new ListNode();
        ListNode p = result;
        while(l1 != null || l2 != null) {
            int num1 = l1 == null ? 0 : l1.val;
            int num2 = l2 == null ? 0 : l2.val;
            p.next = new ListNode(num1 + num2);
            if(l1 != null) {
                l1 = l1.next;
            }
            if(l2 != null) {
                l2 = l2.next;
            }
            p = p.next;
        }
        int carry = 0;
        p = result;
        while(p.next != null) {
            int countNum = carry + p.next.val;
            carry = 0;
            if(countNum <= 9) {
                p.next.val = countNum;
            } else {
                carry = countNum / 10;
                p.next.val = countNum - 10 * carry;
            }
            p = p.next;
        }
        if(carry != 0) {
            p.next = new ListNode(carry);
        }
        return result.next;
    }
}
```

解法二：其实可以在合并的时候直接顺便进行进位的处理，就只需要遍历一遍链表即可

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode result = new ListNode();  
        ListNode p = result;        
        int carry = 0;                  

        while (l1 != null || l2 != null || carry != 0) {
            int num1 = (l1 != null) ? l1.val : 0;
            int num2 = (l2 != null) ? l2.val : 0;
            
            int sum = num1 + num2 + carry;
            carry = sum / 10;             
            p.next = new ListNode(sum % 10);  
            p = p.next;       
            if (l1 != null) l1 = l1.next;
            if (l2 != null) l2 = l2.next;
        }
        return result.next;  
    }
}
```



# 搜索二维矩阵

## 题目描述

编写一个高效的算法来搜索 `*m* x *n*` 矩阵 `matrix` 中的一个目标值 `target` 。该矩阵具有以下特性：

- 每行的元素从左到右升序排列。
- 每列的元素从上到下升序排列。

**示例 1：**

![223](../images/223.jpg)

```
输入：matrix = [[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],[10,13,14,17,24],[18,21,23,26,30]], target = 5
输出：true
```

**示例 2：**

![224](../images/224.jpg)

```
输入：matrix = [[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],[10,13,14,17,24],[18,21,23,26,30]], target = 20
输出：false
```



## 题解

我除了暴力拆解没什么其他思路，看了网上的一种解法直接给跪了，纯妙来的

如下图所示，我们将矩阵逆时针旋转 45° ，并将其转化为图形式，发现其类似于 二叉搜索树 ，即对于每个元素，其左分支元素更小、右分支元素更大。因此，通过从 “根节点” 开始搜索，遇到比 target 大的元素就向左，反之向右，即可找到目标值 target 。

![225](../images/225.png)

这样我们就可以类似于二叉搜索树的方式去进行依次搜索，当遍历完成还没有找到判断没有该数字

**代码实现**

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int i = 0;
        int j = matrix[0].length - 1;
        while(i < matrix.length && j >= 0) {
            if(matrix[i][j] == target) {
                return true;
            } else if(target < matrix[i][j]) {
                j--;
            } else {
                i++;
            }
        }
        return false;
    }
}
```

