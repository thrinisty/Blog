---
title: LeetCode刷题（删除链表的倒数第N个节点、启云方笔试、零钱兑换）
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



# 零钱兑换

## 题目描述

给你一个整数数组 `coins` ，表示不同面额的硬币；以及一个整数 `amount` ，表示总金额。

计算并返回可以凑成总金额所需的 **最少的硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。

你可以认为每种硬币的数量是无限的。

**示例 1：**

```
输入：coins = [1, 2, 5], amount = 11
输出：3 
解释：11 = 5 + 5 + 1
```

**示例 2：**

```
输入：coins = [2], amount = 3
输出：-1
```

**示例 3：**

```
输入：coins = [1], amount = 0
输出：0
```



## 题解

解法一 回溯法（超时）

我一看到这一题，我就想这不是做过嘛，回溯法直接秒，遍历一下之前的结果即可，果不其然时间效率惨淡（39/189）

```java
class Solution {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> list = new ArrayList<>();
    public int coinChange(int[] coins, int amount) {
        dfs(coins, amount, 0);
        if(result.size() == 0) {
            return -1;
        }
        int target = Integer.MAX_VALUE;
        for(int i = 0; i < result.size(); i++) {
            List<Integer> currentList = result.get(i);
            target = Math.min(target, currentList.size());
        }
        return target;
    }

    public void dfs(int[] coins, int amount, int index) {
        if(index == coins.length) {
            return;
        }
        if(amount == 0) {
            result.add(new ArrayList<>(list));
            return;
        }
        dfs(coins, amount, index + 1);
        int newNum = coins[index];
        int newTarget = amount - newNum;
        if(newNum <= amount) {
            list.add(newNum);
            dfs(coins, newTarget, index);
            list.remove(list.size() - 1);
        }
    }
}
```

看了下题解应该使用动态规划完成，不然在不限制硬币数量的时候，复杂度随amount指数增长，就太夸张了

解法二：动态规划

建立一个amount + 1大小的int数组其中存放到这个下表的最小硬币数量，逐个遍历，对每个目标i循环中遍历coins数组，当不超过i时，更新min最小硬币数量（用dp[i - coins[i]] + 1进行迭代min）

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        if(coins.length == 0) {
            return -1;
        }
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, Integer.MAX_VALUE);
        dp[0] = 0;
        for(int i = 1; i <= amount; i++) {
            int min = Integer.MAX_VALUE;
            for(int coin : coins) {
                if(i >= coin && dp[i - coin] != Integer.MAX_VALUE) {
                    min = Math.min(min, dp[i - coin] + 1);
                }
            }
            dp[i] = min;
        }
        return dp[amount] == Integer.MAX_VALUE ? -1 : dp[amount];
    }
}
```

