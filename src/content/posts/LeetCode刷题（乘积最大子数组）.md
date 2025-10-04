---
title: LeetCode刷题（乘积最大子数组）
published: 2025-10-04
updated: 2025-10-04
description: ''
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 乘积最大子数组

## 题目描述

给你一个整数数组 `nums` ，请你找出数组中乘积最大的非空连续 子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

测试用例的答案是一个 **32-位** 整数。

**示例 1:**

```
输入: nums = [2,3,-2,4]
输出: 6
解释: 子数组 [2,3] 有最大乘积 6。
```

**示例 2:**

```
输入: nums = [-2,0,-1]
输出: 0
解释: 结果不能为 2, 因为 [-2,-1] 不是子数组。
```



## 题解

解法一：暴力拆解

第一个想到的就是这个，没想到竟然没超时

```java
class Solution {
    public int maxProduct(int[] nums) {
        int result = Integer.MIN_VALUE;
        for(int i = 0; i < nums.length; i++) {
            int max = nums[i];
            int mul = nums[i]; 
            for(int j = i + 1; j < nums.length; j++) {
                mul *= nums[j];
                max = Math.max(max, mul);
            }
            result = Math.max(result, max);
        }
        return result;
    }
}
```

解法二：LeetCode上一位佬写出来的，搬来用下

设立min变量和max变量，初始化为1，遍历过程中如果数字为负数就交换二者，因为×上负号大的就变成了小的。

迭代一下min和max，每次循环最后再用max迭代result

```
遍历数组时计算当前最大值，不断更新
令imax为当前最大值，则当前最大值为 imax = max(imax * nums[i], nums[i])
由于存在负数，那么会导致最大的变最小的，最小的变最大的。因此还需要维护当前最小值imin，imin = min(imin * nums[i], nums[i])
当负数出现时则imax与imin进行交换再进行下一步计算
时间复杂度：O(n)
```

```java
class Solution {
    public int maxProduct(int[] nums) {
        int result = Integer.MIN_VALUE;
        int max = 1;
        int min = 1;
        for(int num : nums) {
            if(num < 0) {
                int temp = max;
                max = min;
                min = temp;
            }
            max = Math.max(max * num, num);
            min = Math.min(min * num, num);
            result = Math.max(result, max);
        }
        return result;
    }
}
```

