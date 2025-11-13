---
title: LeetCode刷题（相同的树、构造二叉树、根叶和、异位词）
published: 2025-11-10
updated: 2025-11-10
description: '相同的树、构造二叉树、从根节点到叶结点的数字之和、找到字符串中所有字母异位词'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 相同的树

## 题目描述

给你两棵二叉树的根节点 `p` 和 `q` ，编写一个函数来检验这两棵树是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

**示例 1：**

![352](../images/352.jpg)

```
输入：p = [1,2,3], q = [1,2,3]
输出：true
```



## 题解

递归伟大无需多言

```java
class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        if(p == null && q == null) {
            return true;
        }
        if(p == null && q != null) {
            return false;
        }
        if(p != null && q == null) {
            return false;
        }
        if(p.val != q.val) {
            return false;
        }
        return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
    }
}
```



# 构造二叉树

## 题目描述

给定两个整数数组 `inorder` 和 `postorder` ，其中 `inorder` 是二叉树的中序遍历， `postorder` 是同一棵树的后序遍历，请你构造并返回这颗 *二叉树* 。

**示例 1:**

```
输入：inorder = [9,3,15,20,7], postorder = [9,15,7,20,3]
输出：[3,9,20,null,null,15,7]
```

**示例 2:**

```
输入：inorder = [-1], postorder = [-1]
输出：[-1]
```

 

## 题解

和之前的先序构造一样，调整下下标即可，对于根节点的左右两侧的部分是更左右两侧的后序排列

```java
class Solution {
    public TreeNode buildTree(int[] inorder, int[] postorder) {
        int n = inorder.length;
        if(n == 0) {
            return null;
        }
        int value = postorder[n - 1];
        TreeNode root = new TreeNode(value);
        for(int i = 0; i < n; i++) {
            if(inorder[i] == value) {
                int[] leftInorder = Arrays.copyOfRange(inorder, 0, i);
                int[] leftPostorder = Arrays.copyOfRange(postorder, 0, i);
                root.left = buildTree(leftInorder, leftPostorder);
                int[] rightInorder = Arrays.copyOfRange(inorder, i + 1, n);
                int[] rightPostorder = Arrays.copyOfRange(postorder, i, n - 1);
                root.right = buildTree(rightInorder, rightPostorder);
                break;
            }
        }
        return root;
    }
}
```



# 从根节点到叶结点的数字之和

## 题目描述

给你一个二叉树的根节点 `root` ，树中每个节点都存放有一个 `0` 到 `9` 之间的数字。

每条从根节点到叶节点的路径都代表一个数字：

- 例如，从根节点到叶节点的路径 `1 -> 2 -> 3` 表示数字 `123` 。

计算从根节点到叶节点生成的 **所有数字之和** 。

**叶节点** 是指没有子节点的节点。

**示例 1：**

![353](../images/353.jpg)

```
输入：root = [1,2,3]
输出：25
解释：
从根到叶子节点路径 1->2 代表数字 12
从根到叶子节点路径 1->3 代表数字 13
因此，数字总和 = 12 + 13 = 25
```

**示例 2：**

![354](../images/354.jpg)

```
输入：root = [4,9,0,5,1]
输出：1026
解释：
从根到叶子节点路径 4->9->5 代表数字 495
从根到叶子节点路径 4->9->1 代表数字 491
从根到叶子节点路径 4->0 代表数字 40
因此，数字总和 = 495 + 491 + 40 = 1026
```



## 题解

当判断当前节点为叶子节点的时候加上当前值，如果还有则继续遍历左右节点，func额外传入一个count参数用于记录之前的累加和

```java
class Solution {
    int sum = 0;
    public int sumNumbers(TreeNode root) {
        func(root, 0);
        return sum;
    }

    public void func(TreeNode root, int count) {
        if(root == null) {
            return;
        }
        count = count * 10 + root.val;
        if (root.left == null && root.right == null) {
            sum += count;
            return;
        }
        // 非叶子节点继续递归
        func(root.left, count);
        func(root.right, count);
    }
}
```



# 找到字符串中所有字母异位词

## 题目描述

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

**示例 1:**

```
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。
```

 **示例 2:**

```
输入: s = "abab", p = "ab"
输出: [0,1,2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。
```



## 题解

运用滑动窗口以及sort排序依次比较，竟然没有超时，测试用例还是太温柔了

本来我还打算优化一下，如果移除的元素等于加入的元素就不用再次判断了

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> list = new ArrayList<>();
        int n = s.length();
        int m = p.length();
        char[] arr = p.toCharArray();
        Arrays.sort(arr);
        String target = new String(arr);
        for(int i = 0; i < n - m + 1; i++) {
            String str = s.substring(i, i + m);
            char[] charArray = str.toCharArray();
            Arrays.sort(charArray);
            String cur = new String(charArray);
            if(cur.equals(target)) {
                list.add(i);
            }
        }
        return list;
    }
}
```

在判断是异位词的时候while循环看移除移入的元素是否相等相等则可以判断后面的任然是异位词加入集合，之后i++

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> list = new ArrayList<>();
        int n = s.length();
        int m = p.length();
        char[] arr = p.toCharArray();
        Arrays.sort(arr);
        String target = new String(arr);
        for(int i = 0; i < n - m + 1; i++) {
            String str = s.substring(i, i + m);
            char[] charArray = str.toCharArray();
            Arrays.sort(charArray);
            String cur = new String(charArray);
            if(cur.equals(target)) {
                list.add(i);
                while(i + m < n && s.charAt(i) == s.charAt(i + m)) {
                    list.add(++i);
                }
            }
        }
        return list;
    }
}
```

