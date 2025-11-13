---
title: LeetCode刷题（分割等和子集、目标和）
published: 2025-11-12
updated: 2025-11-12
description: ''
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 分割等和子集

## 题目描述

给你一个 **只包含正整数** 的 **非空** 数组 `nums` 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

**示例 1：**

```
输入：nums = [1,5,11,5]
输出：true
解释：数组可以分割成 [1, 5, 5] 和 [11] 。
```

**示例 2：**

```
输入：nums = [1,2,3,5]
输出：false
解释：数组不能分割成两个元素和相等的子集。
```



## 题解

分解为01背包问题，求nums和，如果不为偶数则不可能等和，我们此时从数组中取出和的一半剩余的就是相等的另一半

再通过01背包运用动态规划的方式迭代dp，其中是是否可以从数组中取出对应的和

```java
class Solution {
    public boolean canPartition(int[] nums) {
        int sum = 0;
        for(int num : nums) {
            sum += num;
        }
        if(sum % 2 != 0) return false;
        int target = sum / 2;
        boolean[] dp = new boolean[target + 1];
        dp[0] = true;
        for(int num : nums) {
            for(int i = target; i >= num; i--) {
                dp[i] = dp[i] || dp[i - num];
            }
        }
        return dp[target];
    }
}
```



# 目标和

## 题目描述

给你一个非负整数数组 `nums` 和一个整数 `target` 。

向数组中的每个整数前添加 `'+'` 或 `'-'` ，然后串联起所有整数，可以构造一个 **表达式** ：

- 例如，`nums = [2, 1]` ，可以在 `2` 之前添加 `'+'` ，在 `1` 之前添加 `'-'` ，然后串联起来得到表达式 `"+2-1"` 。

返回可以通过上述方法构造的、运算结果等于 `target` 的不同 **表达式** 的数目。

**示例 1：**

```
输入：nums = [1,1,1,1,1], target = 3
输出：5
解释：一共有 5 种方法让最终目标和为 3 。
-1 + 1 + 1 + 1 + 1 = 3
+1 - 1 + 1 + 1 + 1 = 3
+1 + 1 - 1 + 1 + 1 = 3
+1 + 1 + 1 - 1 + 1 = 3
+1 + 1 + 1 + 1 - 1 = 3
```

**示例 2：**

```
输入：nums = [1], target = 1
输出：1
```



## 题解

没怎么看懂转化为的01背包，待定，后面再看看，先背为敬

```java
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        int sum = 0;
        for (int num : nums)
            sum += num;
        int dif = sum - target;
        if(dif < 0 || dif % 2 != 0) {
            return 0;
        }
        int neg = dif / 2;
        int[] dp = new int[neg + 1];
        dp[0] = 1;
        for(int num : nums) {
            for(int i = neg; i >= num; i--) {
                dp[i] += dp[i - num];
            }
        }
        return dp[neg];
    }
}
```



