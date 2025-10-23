---
title: LeetCode刷题（扔鸡蛋、最长的递增子序列个数）
published: 2025-10-20
updated: 2025-10-20
description: ''
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 扔鸡蛋

## 题目描述

给你 `k` 枚相同的鸡蛋，并可以使用一栋从第 `1` 层到第 `n` 层共有 `n` 层楼的建筑。

已知存在楼层 `f` ，满足 `0 <= f <= n` ，任何从 **高于** `f` 的楼层落下的鸡蛋都会碎，从 `f` 楼层或比它低的楼层落下的鸡蛋都不会破。

每次操作，你可以取一枚没有碎的鸡蛋并把它从任一楼层 `x` 扔下（满足 `1 <= x <= n`）。如果鸡蛋碎了，你就不能再次使用它。如果某枚鸡蛋扔下后没有摔碎，则可以在之后的操作中 **重复使用** 这枚鸡蛋。

请你计算并返回要确定 `f` **确切的值** 的 **最小操作次数** 是多少？

**示例 1：**

```
输入：k = 1, n = 2
输出：2
解释：
鸡蛋从 1 楼掉落。如果它碎了，肯定能得出 f = 0 。 
否则，鸡蛋从 2 楼掉落。如果它碎了，肯定能得出 f = 1 。 
如果它没碎，那么肯定能得出 f = 2 。 
因此，在最坏的情况下我们需要移动 2 次以确定 f 是多少。 
```

**示例 2：**

```
输入：k = 2, n = 6
输出：3
```

**示例 3：**

```
输入：k = 3, n = 14
输出：4
```



## 题解

设置一个dp二维数组，下表为i，j代表，尝试i次，j个鸡蛋可以最多找到几层楼确切的临界值

从1次开始，依次计算i次j个鸡蛋可以选出最高几层楼的情况，直到最高的楼层大于等于目标楼层

```java
class Solution {
    public int superEggDrop(int k, int n) {
        int[][] dp = new int[n + 1][k + 1];
        int i = 1;
        while(true) {
            for(int j = 1; j <= k; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i - 1][j - 1] + 1;
            }
            if(dp[i][k] >= n) {
                return i;
            }
            i++;
        }
    }
}
```



# 最长递增子序列个数

## 题目描述

给定一个未排序的整数数组 `nums` ， *返回最长递增子序列的个数* 。

**注意** 这个数列必须是 **严格** 递增的。

**示例 1:**

```
输入: [1,3,5,4,7]
输出: 2
解释: 有两个最长递增子序列，分别是 [1, 3, 4, 7] 和[1, 3, 5, 7]。
```

**示例 2:**

```
输入: [2,2,2,2,2]
输出: 5
解释: 最长递增子序列的长度是1，并且存在5个子序列的长度为1，因此输出5。
```



## 题解

之前做过一道关于寻找最长递增子序列长度的题目，在它的基础上维护一个count数组，代表包含当前数序列的数字个数，dp对应的max时加上其可能的情况

```java
class Solution {
    public int findNumberOfLIS(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        int[] count = new int[n];
        Arrays.fill(dp, 1);
        Arrays.fill(count, 1);
        int max = 0;
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < i; j++) {
                if(nums[j] < nums[i]) {
                    if(dp[j] + 1 > dp[i]) {
                        dp[i] = dp[j] + 1;
                        count[i] = count[j];
                    } else if(dp[j] + 1 == dp[i]) {
                        count[i] += count[j];
                    }
                }
            }
            max = Math.max(max, dp[i]);
        }
        int result = 0;
        for(int i = 0; i < n; i++) {
            if(dp[i] == max) {
                result += count[i];
            }
        }
        return result;
    }
}
```

