---
title: LeetCode刷题（括号生成，二叉树最大深度）
published: 2025-05-21
updated: 2025-05-21
description: '括号生成，二叉树最大深度'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 括号生成

## 题目描述

数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

**示例 1：**

```
输入：n = 3
输出：["((()))","(()())","(())()","()(())","()()()"]
```

**示例 2：**

```
输入：n = 1
输出：["()"]
```

## 题解

解法一：用字符串存储结果

利用递归解决，用left与right记录剩余的左右括号，当左括号多余右括号时才允许添加右括号（合法）

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        fun(result, "", n, n);
        return result;
    }

    public void fun(List<String> result, String str, int left, int right) {
        if(left == 0 && right == 0) {
            result.add(str);
        }
        if(left > 0) {
            fun(result, str + "(", left - 1, right);
        }
        if(left < right) {
            fun(result, str + ")", left, right - 1);
        }
    }
}
```



解法二：StringBuilder存储

更新以下用StringBuilder存储的解法，因为不用像str一样每一次创建字符串，在内存和速度上都要由于上一种解法，这种解法需要使用到回溯

```java
class Solution {
    List<String> result = new ArrayList<>();
    public List<String> generateParenthesis(int n) {
        StringBuilder str = new StringBuilder();
        fun(str, n, n);
        return result;
    }

    public void fun(StringBuilder str, int left, int right) {
        if(left == 0 && right == 0) {
            result.add(str.toString());
            return;
        }
        if(left != 0) {
            str.append('(');
            fun(str, left - 1, right);
            str.deleteCharAt(str.length() - 1);
        }
        if(left < right) {
            str.append(')');
            fun(str, left, right - 1);
            str.deleteCharAt(str.length() - 1);
        }
    }
}
```



# 二叉树最大深度

## 题目描述

给定一个二叉树 `root` ，返回其最大深度。

二叉树的 **最大深度** 是指从根节点到最远叶子节点的最长路径上的节点数。

 

**示例 1：**

![160](../images/160.jpg)

```
输入：root = [3,9,20,null,null,15,7]
输出：3
```

**示例 2：**

```
输入：root = [1,null,2]
输出：2
```

 

## 题解

简单的分治即可实现，注意终止条件的判断，以及返回0

```java
class Solution {
    public int maxDepth(TreeNode root) {
        if(root == null) {
            return 0;
        }
        int leftMax = maxDepth(root.left);
        int rightMax = maxDepth(root.right);
        return Math.max(leftMax, rightMax) + 1;
    }
}
```

