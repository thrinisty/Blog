---
title: LeetCode刷题（二分查找改编、搜索旋转排序数组）
published: 2025-09-30
updated: 2025-09-30
description: '在排序数组中查找元素第一个位置和最后一个位置、搜索旋转排序数组'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 查找元素第一个和最后一个位置

## 题目描述

给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

你必须设计并实现时间复杂度为 `O(log n)` 的算法解决此问题。

**示例 1：**

```
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]
```

**示例 2：**

```
输入：nums = [5,7,7,8,8,10], target = 6
输出：[-1,-1]
```

**示例 3：**

```
输入：nums = [], target = 0
输出：[-1,-1]
```



## 题解

基础的二分查找

```java
public int search(int[] nums, int left, int right, int target) {
        if(left > right) {
            return -1;
        }
        int mid = (left + right) / 2;
        if(nums[mid] == target) {
            return mid;
        } else if(target < nums[mid]) {
            return search(nums, left, mid - 1, target);
        } else {
            return search(nums, mid + 1, right, target);
        }
    }
```

在基础的二分查找法上修改，找到结果后再次排除该结果，对于最小下表，再查其左边的数组看是否有更小的，右侧反之。

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int left = searchLeft(nums, 0, nums.length - 1, target);
        if(left == -1) {return new int[]{-1, -1};}
        int right = searchRight(nums, 0, nums.length - 1, target);
        return new int[]{left, right};
    }

    public int searchLeft(int[] nums, int left, int right, int target) {
        if(left > right) {
            return -1;
        }
        int mid = (left + right) / 2;
        if(nums[mid] == target) {
            int leftResult = searchLeft(nums, left, mid - 1, target);
            return leftResult == -1 ? mid : leftResult;
        } else if(target < nums[mid]) {
            return searchLeft(nums, left, mid - 1, target);
        } else {
            return searchLeft(nums, mid + 1, right, target);
        }
    }

    public int searchRight(int[] nums, int left, int right, int target) {
        if(left > right) {
            return -1;
        }
        int mid = (left + right) / 2;
        if(nums[mid] == target) {
            int rightResult = searchRight(nums, mid + 1, right, target);
            return rightResult == -1 ? mid : rightResult;
        } else if(target < nums[mid]) {
            return searchRight(nums, left, mid - 1, target);
        } else {
            return searchRight(nums, mid + 1, right, target);
        }
    }
}
```

# 搜索旋转排序数组

## 题目描述

整数数组 `nums` 按升序排列，数组中的值 **互不相同** 。

在传递给函数之前，`nums` 在预先未知的某个下标 `k`（`0 <= k < nums.length`）上进行了 **向左旋转**，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标 **从 0 开始** 计数）。例如， `[0,1,2,4,5,6,7]` 下标 `3` 上向左旋转后可能变为 `[4,5,6,7,0,1,2]` 。

给你 **旋转后** 的数组 `nums` 和一个整数 `target` ，如果 `nums` 中存在这个目标值 `target` ，则返回它的下标，否则返回 `-1` 。

你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。

**示例 1：**

```
输入：nums = [4,5,6,7,0,1,2], target = 0
输出：4
```

**示例 2：**

```
输入：nums = [4,5,6,7,0,1,2], target = 3
输出：-1
```

**示例 3：**

```
输入：nums = [1], target = 0
输出：-1
```



## 题解

二分数组，总有一个子数组是单调的，先判断出哪个部分单调，

```java
class Solution {
    public int search(int[] nums, int target) {
        return searchFunc(nums, 0, nums.length - 1, target);
    }

    public int searchFunc(int[] nums, int left, int right, int target) {
        if(left > right) {
            return -1;
        }
        int mid = left + (right - left) / 2;
        if(nums[mid] == target) {
            return mid;
        }
        if(nums[left] <= nums[mid]) {
            if(target >= nums[left] && target < nums[mid]) {
                return searchFunc(nums, left, mid - 1, target);
            } else {
                return searchFunc(nums, mid + 1, right, target);
            }
        } else {
            if(target > nums[mid] && target <= nums[right]) {
                return searchFunc(nums, mid + 1, right, target);
            } else {
                return searchFunc(nums, left, mid - 1, target);
            }
       }
    }
}
```

