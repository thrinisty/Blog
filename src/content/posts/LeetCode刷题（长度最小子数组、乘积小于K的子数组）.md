---
title: LeetCode刷题（长度最小子数组、乘积小于K的子数组）
published: 2025-12-02
updated: 2025-12-02
description: '长度最小子数组、乘积小于K的子数组'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 长度最小子数组

## 题目描述

给定一个含有 `n` 个正整数的数组和一个正整数 `target` **。**

找出该数组中满足其和 `≥ target` 的长度最小的 **连续子数组** `[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度**。**如果不存在符合条件的子数组，返回 `0` 。

**示例 1：**

```
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

**示例 2：**

```
输入：target = 4, nums = [1,4,4]
输出：1
```

**示例 3：**

```
输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0
```



## 题解

滑动窗口的题目做多了感觉都是一个套路，双指针指向，右指针随着循环依次右移，左指针在满足跳的的基础上循环右跳，在循环中迭代结果

这题中当和大于目标的时候，更新result即可

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int result = Integer.MAX_VALUE;
        int left = 0;
        int count = 0;
        for(int right = 0; right < nums.length; right++) {
            count += nums[right];
            while(count >= target) {
                result = Math.min(result, right - left + 1);
                count -= nums[left];
                left++;
            }
        }
        return result == Integer.MAX_VALUE ? 0 : result;
    }
}
```



# 乘积小于K的子数组

## 题目描述

给定一个正整数数组 `nums`和整数 `k` ，请找出该数组内乘积小于 `k` 的连续的子数组的个数。

**示例 1：**

```
输入: nums = [10,5,2,6], k = 100
输出: 8
解释: 8 个乘积小于 100 的子数组分别为: [10], [5], [2], [6], [10,5], [5,2], [2,6], [5,2,6]。
需要注意的是 [10,5,2] 并不是乘积小于100的子数组。
```

**示例 2：**

```
输入: nums = [1,2,3], k = 0
输出: 0
```



## 题解

还是用滑动窗口的方式，只不过因为需要计算满足的数量，结果不能只是++，而是加上满足条件时两个指针中间的数量和（包括两端）例如k = 10 的时候 left和right指向 （1， 2， 4）这个区间，可以作为结果的是124 24 和4，right - left + 1 = 3

```java
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        int n = nums.length;
        int count = 1;
        int result = 0;
        int left = 0;
        for(int right = 0; right < n; right++) {
            count *= nums[right];
            while(count >= k && left <= right) {
                count /= nums[left];
                left++;
            }
            result += right - left + 1;
        }
        return result;
        
    }
}
```

