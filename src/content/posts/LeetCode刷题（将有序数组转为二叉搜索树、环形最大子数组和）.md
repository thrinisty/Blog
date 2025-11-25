---
title: LeetCode刷题（将有序数组转为二叉搜索树、环形最大子数组和）
published: 2025-11-18
updated: 2025-11-18
description: '将有序数组转为二叉搜索树、环形最大子数组和'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 将有序数组转为二叉搜索树

## 题目描述

给你一个整数数组 `nums` ，其中元素已经按 **升序** 排列，请你将其转换为一棵 平衡 二叉搜索树。

 

**示例 1：**

![381](../images/381.jpg)

```
输入：nums = [-10,-3,0,5,9]
输出：[0,-3,9,-10,null,5]
解释：[0,-10,5,null,-3,null,9] 也将被视为正确答案：
```

**示例 2：**

![382](../images/382.jpg)

```
输入：nums = [1,3]
输出：[3,1]
解释：[1,null,3] 和 [3,1] 都是高度平衡二叉搜索树。
```



## 题解

使用递归求解，对于mid左右侧的数组分别是小于/大于mid的有序数组

```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        return func(nums, 0, nums.length - 1);
    }

    public TreeNode func(int[] nums, int left, int right) {
        if(left > right) return null;
        int mid = left + (right - left) / 2;
        TreeNode root = new TreeNode(nums[mid]);
        root.left = func(nums, left, mid - 1);
        root.right = func(nums, mid + 1, right);
        return root;
    }
}
```



# 环形最大子数组和

## 题目描述

给定一个长度为 `n` 的**环形整数数组** `nums` ，返回 *`nums` 的非空 **子数组** 的最大可能和* 。

**环形数组** 意味着数组的末端将会与开头相连呈环状。形式上， `nums[i]` 的下一个元素是 `nums[(i + 1) % n]` ， `nums[i]` 的前一个元素是 `nums[(i - 1 + n) % n]` 。

**子数组** 最多只能包含固定缓冲区 `nums` 中的每个元素一次。形式上，对于子数组 `nums[i], nums[i + 1], ..., nums[j]` ，不存在 `i <= k1, k2 <= j` 其中 `k1 % n == k2 % n` 。

**示例 1：**

```
输入：nums = [1,-2,3,-2]
输出：3
解释：从子数组 [3] 得到最大和 3
```

**示例 2：**

```
输入：nums = [5,-3,5]
输出：10
解释：从子数组 [5,5] 得到最大和 5 + 5 = 10
```

**示例 3：**

```
输入：nums = [3,-2,2,-3]
输出：3
解释：从子数组 [3] 和 [3,-2,2] 都可以得到最大和 3
```



## 题解

题解写注解里了

```java
class Solution {
    public int maxSubarraySumCircular(int[] nums) {
        int n = nums.length;
        int[] leftMax = new int[n];//记录从左侧头开始最大和
        leftMax[0] = nums[0];
    
        int sum = nums[0];
        int result = nums[0];
        int leftSum = nums[0];//用于记录左侧的累加和
        for(int i = 1; i < n; i++) {
            //计算从左侧开始的累加和
            leftSum += nums[i];
            //计算从左侧开始到该位置的最大可能
            leftMax[i] = Math.max(leftMax[i - 1], leftSum);
            //以下代表不一定要从头开始的选取方式,思路和子数组最大和一致
            sum = Math.max(sum + nums[i], nums[i]);
            result = Math.max(sum, result);
        }

        int rightSum = 0;//记录从右侧开始的累加和
        for(int i = n - 1; i > 0; i--) {
            rightSum += nums[i];
            //右侧该位置的累加和加上从最左侧开始的最大和，用结果迭代result
            result = Math.max(result, rightSum + leftMax[i - 1]);
        }
        return result;
    }
}
```

