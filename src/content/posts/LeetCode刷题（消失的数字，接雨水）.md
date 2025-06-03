---
title: LeetCode刷题（消失的数字，接雨水）
published: 2025-06-03
updated: 2025-06-03
description: '消失的数字，接雨水'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 消失的数字

## 题目描述

给你一个含 `n` 个整数的数组 `nums` ，其中 `nums[i]` 在区间 `[1, n]` 内。请你找出所有在 `[1, n]` 范围内但没有出现在 `nums` 中的数字，并以数组的形式返回结果。

 

**示例 1：**

```
输入：nums = [4,3,2,7,8,2,3,1]
输出：[5,6]
```

**示例 2：**

```
输入：nums = [1,1]
输出：[2]
```

## 题解

简单题我重拳出击，但自己写的没有达到n（0）的空间复杂度

解法一：通过一个辅助数组记录出现过的数字，设置为true，再次遍历辅助数组，当对于数组为false的下表加入list集合

```java
class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> list = new ArrayList<>();
        int n = nums.length;
        boolean[] flags = new boolean[n + 1];
        for(int i = 0; i < n; i++) {
            flags[nums[i]] = true;
        }
        for(int i = 1; i <= n; i++) {
            if(flags[i] == false) {
                list.add(i);
            }
        }
        return list;
    }
}
```



解法二：在原地实现

1.遍历每个元素，对索引进行标记将对应索引位置的值变为负数；
2.遍历下索引，看看哪些索引位置上的数不是负数的，位置上不是负数的索引，对应的元素就是不存在的。

```java
class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> list = new ArrayList<>();
        int len = nums.length;
        for(int i = 0; i < len; i++) {
            int num = Math.abs(nums[i]);
            int idx = num - 1;
            if(nums[idx] > 0) {
                nums[idx] *= -1;
            }
        }
        for(int i = 0; i < len; i++) {
            if(nums[i] > 0) {
                int num = i + 1;
                list.add(num);
            }
        }
        return list;
    }
}
```

其实本质上就是将第一种解法中的辅助数组，换为了在原数组上的下表用 - 表示为true，在不影响原数组的时候的情况下记录信息

# 接雨水

## 题目描述

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

 

**示例 1：**

![179](../images/179.png)

```
输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6
解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。 
```

**示例 2：**

```
输入：height = [4,2,0,3,2,5]
输出：9
```



## 题解

按照每一列进行求值，分别找出每一列左右两侧最高的值，取二者最小，如果最小的值小于或等于当前列高度，则不能储水，反之当前列可以存min - height[i]单位的水

```java
class Solution {
    public int trap(int[] height) {
        int sum = 0;
        int n = height.length;
        for(int i = 1; i < n - 1; i++) {
            int max_left = 0;
            for(int j = 0; j < i; j++) {
                if(height[j] > max_left) {
                    max_left = height[j];
                }
            }
            int max_right = 0;
            for(int j = i + 1; j < n; j++) {
                if(height[j] > max_right) {
                    max_right = height[j];
                }
            }
            int min = Math.min(max_left, max_right);
            if(min > height[i]) {
                sum += min - height[i];
            }
        }
        return sum;
    }
}
```

空间复杂度n^2，对于测试用例的其中一个超时，其实本质上就是没有复用到左右两侧的最高列，以下是一个双指针的解法

```java
class Solution {
    public int trap(int[] height) {
        int n = height.length;
        int sum = 0;

        int left = 0;
        int right = n - 1;
        int leftMax = height[left];
        int rightMax = height[right];
        left++;
        right--;

        while(left <= right) {
            leftMax = Math.max(leftMax, height[left]);
            rightMax = Math.max(rightMax, height[right]);
            if(leftMax < rightMax) {
                sum += leftMax - height[left];
                left++;
            } else {
                sum += rightMax - height[right];
                right--;
            }
        }
        return sum;
    }
}
```

