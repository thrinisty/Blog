---
title: LeetCode刷题（二叉搜索树最小绝对差、搜索树第K小元素）
published: 2025-11-13
updated: 2025-11-13
description: '二叉搜索树最小绝对差、搜索树第K小元素'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 二叉搜索树最小绝对差

## 题目描述

给你一个二叉搜索树的根节点 `root` ，返回 **树中任意两不同节点值之间的最小差值** 。

差值是一个正数，其数值等于两值之差的绝对值。

**示例 1：**

![361](../images/361.jpg)

```
输入：root = [4,2,6,1,3]
输出：1
```

**示例 2：**

![362](../images/362.jpg)

```
输入：root = [1,0,48,null,null,12,49]
输出：1
```



## 题解

用pre记录前一个值，在中序遍历中进行对于result迭代

```java
class Solution {
    int result = Integer.MAX_VALUE;
    int pre = -1;
    public int getMinimumDifference(TreeNode root) {
        dfs(root);
        return result;
    }
    public void dfs(TreeNode root) {
        if(root == null) return;
        dfs(root.left);
        if(pre != -1) {
            result = Math.min(result, Math.abs(pre - root.val));
        }
        pre = root.val;
        dfs(root.right);
    }   
}
```

补一个非递归的方式

```java
class Solution {
    int result = Integer.MAX_VALUE;
    int pre = -1;
    public int getMinimumDifference(TreeNode root) {
        Stack<TreeNode> stack = new Stack();
        TreeNode p = root;
        while(!stack.isEmpty() || p != null) {
            while(p != null) {
                stack.push(p);
                p = p.left;
            }
            TreeNode node = stack.pop();
            if(pre != -1) {
                result = Math.min(result, Math.abs(node.val - pre));
            }
            pre = node.val;
            p = node.right;
        }
        return result;
    }
}
```



# 搜索树第K小元素

## 题目描述

给定一个二叉搜索树的根节点 `root` ，和一个整数 `k` ，请你设计一个算法查找其中第 `k` 小的元素（从 1 开始计数）。

**示例 1：**

![363](../images/363.jpg)

```
输入：root = [3,1,4,null,2], k = 1
输出：1
```

**示例 2：**

![364](../images/364.jpg)

```
输入：root = [5,3,6,2,4,null,null,1], k = 3
输出：3
```



## 题解

用中序遍历，遍历到数字就加一，如果当前数字在第K为就设置当前数字为result

```java
class Solution {
    int num = 1;
    int result = -1;
    int target;
    public int kthSmallest(TreeNode root, int k) {
        target = k;
        dfs(root);
        return result;
    }

    public void dfs(TreeNode root) {
        if(root == null) return; 
        dfs(root.left);
        if(num == target) {
            result = root.val;
        }
        num++;
        dfs(root.right);
    }
}
```

非递归

```java
class Solution {
    public int kthSmallest(TreeNode root, int k) {
        Stack<TreeNode> stack = new Stack();
        TreeNode p = root;
        while(!stack.isEmpty() || p != null) {
            while(p != null) {
                stack.push(p);
                p = p.left;
            }
            TreeNode node = stack.pop();
            k--;
            if(k == 0) {
                return node.val;
            }
            p = node.right;
        }
        return p.val;
    }
}
```

