---
title: LeetCode刷题（删除链表的倒数第N个节点、启云方笔试）
published: 2025-09-28
updated: 2025-09-28
description: '删除链表的倒数第N个节点、二叉树面试题（层序，构造）'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 删除链表的倒数第N个节点

## 题目描述

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

 ![221](../images/221.jpg)

**示例 1：**

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

**示例 2：**

```
输入：head = [1], n = 1
输出：[]
```

**示例 3：**

```
输入：head = [1,2], n = 1
输出：[1]
```

 

## 题解

解法一：

我的思路是遍历一遍链表，找出节点偏移，用一个方法结合偏移删除目标节点

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode p = head;
        int count = 0;
        while(p != null) {
            count++;
            p = p.next;
        }
        if(count == 1) {
            return null;
        }
        int index = count - n;
        return removeNode(head, index);
    }

    public ListNode removeNode(ListNode head, int n) {
        ListNode result = new ListNode();
        result.next = head;
        ListNode p = result;
        for(int i = 0; i < n; i++) {
            p = p.next;
        }
        p.next = p.next.next;
        return result.next;
    }
}
```



解法二：

先将快指针移动n步，之后快慢指针一起移动直到快指针的下一个节点为空，这个时候慢指针所指的下一个元素即为需要删除的节点

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode result = new ListNode();
        result.next = head;
        ListNode fast = result;
        ListNode slow = result;
        for(int i = 0; i < n; i++) {
            fast = fast.next;
        }
        while(fast.next != null) {
            fast = fast.next;
            slow = slow.next;
        }
        slow.next = slow.next.next;
        return result.next;
    }
}
```



# 启云方笔试

## 题目描述

根据后序遍历的结果和中序数组求增序遍历的结果

给定两个整数数组 `postorder` 和 `inorder`，其中 `postorder` 是二叉搜索树（BST）的**后序遍历**结果，`inorder` 是同一棵树的**中序遍历**结果，请你构造并返回这棵二叉搜索树的**层序遍历**结果（即广度优先遍历，按层输出节点值）。

二叉搜索树的定义为：

- 左子树的所有节点值均小于根节点的值。
- 右子树的所有节点值均大于根节点的值。
- 左右子树也必须是二叉搜索树。

#### **示例 1**:

- 输入
  - `postorder = [9, 15, 7, 20, 3]`
  - `inorder = [9, 3, 15, 20, 7]`
- **输出**: `[ [3], [9, 20], [15, 7] ]`
  （层序遍历结果，同层节点从左到右排列）

#### **示例 2**:

- 输入
  - `postorder = [2,1]`
  - `inorder = [2,1]`
- **输出**: `[ [1], [2] ]`



## 题解

其实就是之前先序遍历+中序遍历构造二叉树和层序遍历的综合题目，改成了后序遍历，只需要取数组最后一个节点作为根即可，思路大致一致，注意一下下标

```java
package com.nwpu;

import java.util.*;

class TreeNode {
    char val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(char x) {
        val = x;
    }
}

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String back = scanner.next();
        String mid = scanner.next();
        char[] backArray = back.toCharArray();
        char[] midArray = mid.toCharArray();
        TreeNode root = build(backArray, midArray);
//        func(root);
        printTree(root);
    }

    public static TreeNode build(char[] backArray, char[] midArray) {
        if(backArray.length == 0 || midArray.length == 0) {
            return null;
        }
        char val = backArray[backArray.length - 1];
        TreeNode root = new TreeNode(val);
        for(int i = 0; i < midArray.length; i++) {
            if(midArray[i] == val) {
                char[] leftBackArray = Arrays.copyOfRange(backArray, 0, i);
                char[] leftMidArray = Arrays.copyOfRange(midArray, 0, i);
                char[] rightBackArray = Arrays.copyOfRange(backArray, i, backArray.length - 1);
                char[] rightMidArray = Arrays.copyOfRange(midArray, i + 1, midArray.length);
                root.left = build(leftBackArray, leftMidArray);
                root.right = build(rightBackArray, rightMidArray);
                break;
            }
        }
        return root;
    }

    public static void printTree(TreeNode root) {
        if(root == null) {
            return;
        }
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()) {
            TreeNode node = queue.poll();
            System.out.print(node.val);
            if(node.left != null) {
                queue.offer(node.left);
            }
            if(node.right != null) {
                queue.offer(node.right);
            }
        }
    }

    public static void func(TreeNode root) {
        if(root == null) {
            return;
        }
        func(root.left);
        System.out.println(root.val);
        func(root.right);
    }
}
```



