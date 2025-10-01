---
title: LeetCode刷题（和为K的子数组、二叉树中最大路径和）
published: 2025-10-01
updated: 2025-10-01
description: '和为K的子数组、二叉树中最大路径和'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 和为K的子数组

## 题目描述

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 *该数组中和为 `k` 的子数组的个数* 。

子数组是数组中元素的连续非空序列。

**示例 1：**

```
输入：nums = [1,1,1], k = 2
输出：2
```

**示例 2：**

```
输入：nums = [1,2,3], k = 3
输出：2
```



## 题解

解法一：暴力拆解，这里不break作剪枝是因为有负数和0的测例，同一种开头的i可能有大于1的情况

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int result = 0;
        for(int i = 0; i < nums.length; i++) {
            int sum = 0;
            for(int j = i; j < nums.length; j++) {
                sum += nums[j];
                if(sum == k) {
                    result++;
                }
            }
        }
        return result;
    }
}
```

解法二：前缀和+哈希表

设立一个哈希表依次存储数组的各项前缀和，值为其存在的次数

遍历途中哈希表key如果存在target，代表存在value种情况符合和为k，将value取出加到结果上，最后将当前的前缀和存到map中，如果存在则对应的value++

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int sum = 0;
        int result = 0;
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);
        for(int i = 0; i < nums.length; i++) {
            sum += nums[i];
            int target = sum - k;
            if(map.containsKey(target)) {
                result += map.get(target);
            }
            map.put(sum, map.getOrDefault(sum, 0) + 1);
        } 
        return result;
    }
}
```



# 二叉树中最大路径和

## 题目描述

二叉树中的 **路径** 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 **至多出现一次** 。该路径 **至少包含一个** 节点，且不一定经过根节点。

**路径和** 是路径中各节点值的总和。

给你一个二叉树的根节点 `root` ，返回其 **最大路径和** 。

**示例 1：**

![226](../images/226.jpg)

```
输入：root = [1,2,3]
输出：6
解释：最优路径是 2 -> 1 -> 3 ，路径和为 2 + 1 + 3 = 6
```

**示例 2：**

![227](../images/227.jpg)

```
输入：root = [-10,9,20,null,null,15,7]
输出：42
解释：最优路径是 15 -> 20 -> 7 ，路径和为 15 + 20 + 7 = 42
```



## 题解

求二叉树直径算法

```java
class Solution {
    int result = 0;
    public int diameterOfBinaryTree(TreeNode root) {
        int height = deep(root);
        return result;
    }

    public int deep(TreeNode root) {
        if(root == null) {
            return 0;
        }
        int leftHeight = deep(root.left);
        int rightHeight = deep(root.right);
        result = Math.max(result, leftHeight + rightHeight);
        return Math.max(leftHeight, rightHeight) + 1;
    }
}
```

思路和求最长路径类似，在dfs算法中迭代result结果，只不过返回的结果是最长的一个边，因为有负数的存在还需判断是否不取该root节点的边和（返回0）

```java
class Solution {
    int result = Integer.MIN_VALUE;
    public int maxPathSum(TreeNode root) {
        dfs(root);
        return result;
    }
    public int dfs(TreeNode root) {
        if(root == null) {
            return 0;
        }
        int left = dfs(root.left);
        int right = dfs(root.right);
        result = Math.max(result, left + root.val + right);
        return Math.max(Math.max(left, right) + root.val, 0);
    }
}
```

