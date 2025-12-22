---
title: LeetCode刷题（搜索插入位置、山脉数组的封顶索引）
published: 2025-12-21
updated: 2025-12-21
description: '搜索插入位置、山脉数组的封顶索引'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 搜索插入位置

## 题目描述

给定一个排序的整数数组 `nums` 和一个整数目标值` target` ，请在数组中找到 `target `，并返回其下标。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。

**示例 1：**

```
输入: nums = [1,3,5,6], target = 5
输出: 2
```

**示例 2：**

```
输入: nums = [1,3,5,6], target = 2
输出: 1
```

**示例 3：**

```
输入: nums = [1,3,5,6], target = 7
输出: 4
```

**示例 4：**

```
输入: nums = [1,3,5,6], target = 0
输出: 0
```

**示例 5：**

```
输入: nums = [1], target = 0
输出: 0
```



## 题解

二分查找

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        while(left <= right) {
            int mid = left + (right - left) / 2;
            if(target == nums[mid]) {
                return mid;
            } else if(nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        return left;
    }
}
```



# 山脉数组的封顶索引

## 题目描述

符合下列属性的数组 `arr` 称为 **山峰数组**（**山脉数组）** ：

**示例 1：**

```
输入：arr = [0,1,0]
输出：1
```

**示例 2：**

```
输入：arr = [1,3,5,4,2]
输出：2
```

**示例 3：**

```
输入：arr = [0,10,5,2]
输出：1
```

**示例 4：**

```
输入：arr = [3,4,5,1]
输出：2
```

**示例 5：**

```
输入：arr = [24,69,100,99,79,78,67,36,26,19]
输出：2
```



## 题解

依旧二分查找，只不过左右设置为第2和n - 1个元素（题目中没有最左右是最大的情况）

```java
class Solution {
    public int peakIndexInMountainArray(int[] arr) {
        int left = 1;
        int right = arr.length - 2;
        while(left <= right) {
            int mid = left + (right - left) / 2;
            if(arr[mid] > arr[mid + 1]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        } 
        return left;
    }
}
```

