---
title: LeetCode刷题（二叉搜索树中的中序后继、两数之和Ⅱ）
published: 2025-12-15
updated: 2025-12-15
description: '二叉搜索树中的中序后继、两数之和Ⅱ'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 二叉搜索树中的中序后继

## 题目描述

给定一棵二叉搜索树和其中的一个节点 `p` ，找到该节点在树中的中序后继。如果节点没有中序后继，请返回 `null` 。

节点 `p` 的后继是值比 `p.val` 大的节点中键值最小的节点，即按中序遍历的顺序节点 `p` 的下一个节点。

**示例 1：**

![507](../images/507.png)

```
输入：root = [2,1,3], p = 1
输出：2
解释：这里 1 的中序后继是 2。请注意 p 和返回值都应是 TreeNode 类型。
```

**示例 2：**

![508](../images/508.png)

```
输入：root = [5,3,6,2,4,null,null,1], p = 6
输出：null
解释：因为给出的节点没有中序后继，所以答案就返回 null 了。
```



## 题解

解法一：通过递归中序遍历构造List集合，之后遍历List集合

```java
class Solution {
    List<TreeNode> list = new ArrayList<>();
    public TreeNode inorderSuccessor(TreeNode root, TreeNode p) {
        dfs(root);
        for(int i = 0; i < list.size() - 1; i++) {
            TreeNode node = list.get(i);
            if(node == p) {
                return list.get(i + 1);
            }
        }
        return null;
    }

    public void dfs(TreeNode root) {
        if(root == null) {
            return;
        }
        dfs(root.left);
        list.add(root);
        dfs(root.right);
    }
}
```

解法二：通过栈遍历，并设立flag检测是否遍历到了target取出flag为true时的第一个node

```java
class Solution {
    public TreeNode inorderSuccessor(TreeNode root, TreeNode p) {
        TreeNode node = root;
        Stack<TreeNode> stack = new Stack<>();
        boolean flag = false;
        while(node != null || !stack.isEmpty()) {
            while(node != null) {
                stack.push(node);
                node = node.left;
            }
            node = stack.pop();
            if(flag) {
                return node;
            }
            if(node == p) {
                flag = true;
            }
            node = node.right;
        }
        return null;
    }
}
```



# 两数之和Ⅱ

## 题目描述

给定一个二叉搜索树的 **根节点** `root` 和一个整数 `k` , 请判断该二叉搜索树中是否存在两个节点它们的值之和等于 `k` 。假设二叉搜索树中节点的值均唯一。

**示例 1：**

```
输入: root = [8,6,10,5,7,9,11], k = 12
输出: true
解释: 节点 5 和节点 7 之和等于 12
```

**示例 2：**

```
输入: root = [8,6,10,5,7,9,11], k = 22
输出: false
解释: 不存在两个节点值之和为 22 的节点
```



## 题解

运用栈迭代遍历，以及Set辅助存储，思路和两数之和Ⅰ类似

```java
class Solution {
    public boolean findTarget(TreeNode root, int k) {
        TreeNode p = root;
        Stack<TreeNode> stack = new Stack<>();
        Set<Integer> set = new HashSet<>();
        while(p != null || !stack.isEmpty()) {
            while(p != null) {
                stack.push(p);
                p = p.left;
            }
            p = stack.pop();
            if(set.contains(k - p.val)) {
                return true;
            }
            set.add(p.val);
            p = p.right;
        }
        return false;
    }
}
```

