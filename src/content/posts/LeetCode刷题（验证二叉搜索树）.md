---
title: LeetCode刷题（验证二叉搜索树）
published: 2025-10-03
updated: 2025-10-03
description: '验证二叉搜索树'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 验证二叉搜索树

## 题目描述

给你一个二叉树的根节点 `root` ，判断其是否是一个有效的二叉搜索树。

**有效** 二叉搜索树定义如下：

- 节点的左子树只包含 **严格小于** 当前节点的数。
- 节点的右子树只包含 **严格大于** 当前节点的数。
- 所有左子树和右子树自身必须也是二叉搜索树。

**示例 1：**



```
输入：root = [2,1,3]
输出：true
```

**示例 2：**



```
输入：root = [5,1,4,null,null,3,6]
输出：false
解释：根节点的值是 5 ，但是右子节点的值是 4 。
```



## 题解

递归一手，遍历结点的时候同时传入节点必须小于的值和必须大于的值，对于左节点而言必须小于父节点的值，对于右节点最小的值必须要大于父节点的值

```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        return isValid(root, Integer.MIN_VALUE, Integer.MAX_VALUE);
    }

    public boolean isValid(TreeNode root, int min, int max) {
        if(root == null) {
            return true;
        }
        if(root.val >= max || root.val <= min) {
            return false;
        }
        return isValid(root.left, min, root.val) && isValid(root.right, root.val , max);
    }
}
```

