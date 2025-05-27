---
title: LeetCode刷题（构造二叉树，前缀树）
published: 2025-05-26
updated: 2025-05-26
description: '从前序与中序遍历序列构造二叉树，实现前缀树'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 前序与中序遍历序列构造二叉树

## 题目描述

给定两个整数数组 `preorder` 和 `inorder` ，其中 `preorder` 是二叉树的**先序遍历**， `inorder` 是同一棵树的**中序遍历**，请构造二叉树并返回其根节点。

 

**示例 1:**

![166](../images/166.jpg)

```
输入: preorder = [3,9,20,15,7],inorder = [9,3,15,20,7]
输出: [3,9,20,null,null,15,7]
```

**示例 2:**

```
输入: preorder = [-1], inorder = [-1]
输出: [-1]
```

## 题解

```java
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        if(preorder.length == 0 || inorder.length == 0) {
            return null;
        }
        int root_val = preorder[0];
        TreeNode root = new TreeNode(root_val);
        for(int i = 0; i < preorder.length; i++) {
            if(root_val == inorder[i]) {
                int[] pre_left = Arrays.copyOfRange(preorder, 1, i + 1);
                int[] in_left = Arrays.copyOfRange(inorder, 0, i);
                root.left = buildTree(pre_left, in_left);
                
                int[] pre_right = Arrays.copyOfRange(preorder, i + 1, preorder.length);
                int[] in_right = Arrays.copyOfRange(inorder, i + 1, inorder.length);
                root.right = buildTree(pre_right, in_right); 
                break;
            }   
        }
        return root;
    }
}
```



# 前缀树

## 题目描述

 **前缀树** 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补全和拼写检查。

请你实现 Trie 类：

- `Trie()` 初始化前缀树对象。
- `void insert(String word)` 向前缀树中插入字符串 `word` 。
- `boolean search(String word)` 如果字符串 `word` 在前缀树中，返回 `true`（即，在检索之前已经插入）；否则，返回 `false` 。
- `boolean startsWith(String prefix)` 如果之前已经插入的字符串 `word` 的前缀之一为 `prefix` ，返回 `true` ；否则，返回 `false` 。

**示例：**

```
输入
["Trie", "insert", "search", "search", "startsWith", "insert", "search"]
[[], ["apple"], ["apple"], ["app"], ["app"], ["app"], ["app"]]
输出
[null, null, true, false, true, null, true]

解释
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");   // 返回 True
trie.search("app");     // 返回 False
trie.startsWith("app"); // 返回 True
trie.insert("app");
trie.search("app");     // 返回 True
```

## 题解

相关模板

在理解前缀树的定义之后，实现的思路并不复杂，但和传统的二叉树实现有些不同，创建了一个Node类用于存储树的结构，而并非直接递归的使用Trie

```java
class Node {
        Node[] son = new Node[26];
        boolean end;
    }

class Trie {
    private Node root = new Node();

    public void insert(String word) {
        Node cur = root;
        char[] words = word.toCharArray();
        for(char c : words) {
            int i = c - 'a';
            if(cur.son[i] == null) {
                cur.son[i] = new Node();
            }
            cur = cur.son[i];
        }
        cur.end = true;
    }
    
    public boolean search(String word) {
        return find(word) == 2;
    }
    
    public boolean startsWith(String prefix) {
        return find(prefix) != 0;
    }

    private int find(String word) {
        Node cur = root;
        char[] words = word.toCharArray();
        for(char c : words) {
            int i = c - 'a';
            if(cur.son[i] == null) {
                return 0;
            }
            cur = cur.son[i];
        }
        return cur.end ? 2 : 1;
    }
}
```

