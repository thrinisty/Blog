---
title: LeetCode刷题（买股票一次买入卖出，二叉树直径）
published: 2025-06-06
updated: 2025-06-06
description: '区别于之前的买股票，二叉树直径'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 买股票的最佳时机

## 题目描述

给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0` 。

**示例 1：**

```
输入：[7,1,5,3,6,4]
输出：5
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
```

**示例 2：**

```
输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下, 没有交易完成, 所以最大利润为 0。
```



## 题解

这一道题目我还以为是可以多次买入卖出，只是不像之前的有冷冻期而已，但是只能买入卖出一次

我们记录历史的最低价格，遍历后续元素的时候迭代刷新，并计算当前价格减去历史价格的值是否大于先前的值，迭代之，最后的max即为我们需要的结果

```java
class Solution {
    public int maxProfit(int[] prices) {
        int max = 0;
        int min = Integer.MAX_VALUE;
        for(int price : prices) {
            min = Math.min(min, price);
            max = Math.max(max, price - min);
        }
        return max;
    }
}
```



# 二叉树直径

## 题目描述

给你一棵二叉树的根节点，返回该树的 **直径** 。

二叉树的 **直径** 是指树中任意两个节点之间最长路径的 **长度** 。这条路径可能经过也可能不经过根节点 `root` 。

两节点之间路径的 **长度** 由它们之间边数表示。

 

**示例 1：**

![181](../images/181.jpg)

```
输入：root = [1,2,3,4,5]
输出：3
解释：3 ，取路径 [4,2,1,3] 或 [5,2,1,3] 的长度。
```

**示例 2：**

```
输入：root = [1,2]
输出：1
```



## 题解

递归，取左右字数深度的和迭代结果，返回二叉树深度

其实就是在之前求二叉树深度的基础上迭代result最大结果

```java
class Solution {
    int result = 0;
    public int diameterOfBinaryTree(TreeNode root) {
        maxDepth(root);
        return result;
    }

    private int maxDepth(TreeNode root) {
        if(root == null) {
            return 0;
        }
        int leftMax = maxDepth(root.left);
        int rightMax = maxDepth(root.right);
        int maxLength = leftMax + rightMax;
        result = Math.max(result, maxLength);
        return Math.max(leftMax, rightMax) + 1;
    }
}
```

