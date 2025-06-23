---
title: LeetCode刷题（最大子序列和，打家劫舍数组版本）
published: 2025-06-22
updated: 2025-06-22
description: '最大子序列和，打家劫舍数组版本'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 最大子序列和

## 题目描述

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组**是数组中的一个连续部分。

 

**示例 1：**

```
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
```

**示例 2：**

```
输入：nums = [1]
输出：1
```

**示例 3：**

```
输入：nums = [5,4,-1,7,8]
输出：23
```



## 题解

如果是sum>0的时候，说明之前的结果是有增益的，需要把sum给加上当前遍历到的数字，而sum<=0的时候舍弃前面的结果，而将sum重新赋值当前遍历到的数字，用result存储当前遍历到的最大结果，每次循环最后的末尾用sum迭代之

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int result = nums[0];
        int sum = 0;
        for(int num : nums) {
            if(sum > 0) {
                sum += num;
            } else {
                sum = num;
            }
            result = Math.max(result, sum);
        }
        return result;
    }
}
```

# 打家劫舍数组版本

## 题目描述

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。

给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。

 

**示例 1：**

```
输入：[1,2,3,1]
输出：4
解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
     偷窃到的最高金额 = 1 + 3 = 4 。
```

**示例 2：**

```
输入：[2,7,9,3,1]
输出：12
解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
     偷窃到的最高金额 = 2 + 9 + 1 = 12 。
```



## 题解

之前做过运用二叉树存储结果的一道题目，这一道题目运用的是数组进行存储，仿照类似的解法，用一个长度为2的数组存储结果，在方法的内部递归的调用方法求出打劫下一个房屋的结果集，根据结果集分别给0不打劫的情况，1打劫的情况分别赋值，即可正确地求出结果

```java
class Solution {
    public int rob(int[] nums) {
        int[] result = rotFun(nums, 0);
        return Math.max(result[0], result[1]);
    }

    public int[] rotFun(int[] nums, int index) {
        int[] result = new int[2];
        if(nums.length == index) {
            return result;
        }
        int[] nextTarget = rotFun(nums, index + 1);
        result[0] = Math.max(nextTarget[0], nextTarget[1]);
        result[1] = nums[index] + nextTarget[0];
        return result;
    }
}
```

