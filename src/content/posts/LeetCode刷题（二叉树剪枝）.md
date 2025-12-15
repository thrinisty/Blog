---
title: LeetCode刷题（二叉树剪枝）
published: 2025-12-14
updated: 2025-12-14
description: '二叉树剪枝'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 二叉树剪枝

## 题目描述

给定一个二叉树 **根节点** `root` ，树的每个节点的值要么是 `0`，要么是 `1`。请剪除该二叉树中所有节点的值为 `0` 的子树。

节点 `node` 的子树为 `node` 本身，以及所有 `node` 的后代。

**示例 1：**

![503](../images/503.png)

```
输入: [1,null,0,0,1]
输出: [1,null,0,null,1] 
解释: 
只有红色节点满足条件“所有不包含 1 的子树”。
右图为返回的答案。
```

**示例 2：**

![504](../images/504.png)

```
输入: [1,0,1,0,0,0,1]
输出: [1,null,1,null,1]
解释: 
```

**示例 3：**

![505](../images/505.png)

```
输入: [1,1,0,1,1,0,1,0]
输出: [1,1,0,1,1,null,1]
解释: 
```



## 题解

运用递归求解，当该节点为0，且无左右子树的时候返回null进行剪枝，对节点的左右节点也同样递归进行剪枝

```java
class Solution {
    public TreeNode pruneTree(TreeNode root) {
        if(root == null) {
            return null;
        }
        root.left = pruneTree(root.left);
        root.right = pruneTree(root.right);
        if(root.val == 0 && root.left == null && root.right == null) {
            return null;
        }
        return root;
    }
}
```

