---
title: LeetCode刷题（搜索插入位置、搜索二维矩阵、寻找峰值）
published: 2025-11-19
updated: 2025-11-19
description: '搜索插入位置、搜索二维矩阵、寻找峰值'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 搜索插入位置

## 题目描述

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。

**示例 1:**

```
输入: nums = [1,3,5,6], target = 5
输出: 2
```

**示例 2:**

```
输入: nums = [1,3,5,6], target = 2
输出: 1
```

**示例 3:**

```
输入: nums = [1,3,5,6], target = 7
输出: 4
```



## 题解

解法一：依次遍历

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int n = nums.length;
        for(int i = 0; i < n; i++) {
            if(nums[i] >= target) {
                return i;
            }
        }
        return n;
    }
}
```

解法二：二分查找

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int n = nums.length;
        return search(nums, 0, n - 1, target);
    }

    public int search(int[] nums, int left, int right, int target) {
        if(left > right) {
            return left;
        }
        int mid = left + (right - left) / 2;
        int num = nums[mid];
        if(num == target) {
            return mid;
        } else if(target < num) {
            return search(nums, left, mid - 1, target);
        } else {
            return search(nums, mid + 1, right, target);
        }
    }
}
```

解法三：补一个循环非递归

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int n = nums.length;
        int left = 0;
        int right = n - 1;
        while(left <= right) {
            int mid = (left + right) / 2;
            int num = nums[mid];
            if(num == target) {
                return mid;
            } else if(target < num) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return left;
    }
}
```



# 搜索二维矩阵

## 题目描述

给你一个满足下述两条属性的 `m x n` 整数矩阵：

- 每行中的整数从左到右按非严格递增顺序排列。
- 每行的第一个整数大于前一行的最后一个整数。

给你一个整数 `target` ，如果 `target` 在矩阵中，返回 `true` ；否则，返回 `false` 。

 

**示例 1：**

![387](../images/387.jpg)

```
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
输出：true
```

**示例 2：**

![388](../images/388.jpg)

```
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 13
输出：false
```



## 题解

解法一：先找出目标元素在哪一行，然后对于行二分查找

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int row = matrix.length;
        int col = matrix[0].length;
        int targetRow = -1;
        for(int i = 0; i < row; i++) {
            if(matrix[i][0] <= target && target <= matrix[i][col - 1]) {
                targetRow = i;
                break;
            }
        }
        if(targetRow == -1) return false;
        int[] searchRow = matrix[targetRow];
        int left = 0;
        int right = col - 1;
        while(left <= right) {
            int mid = left + (right - left) / 2;
            int num = searchRow[mid];
            if(num == target) {
                return true;
            } else if(target < num) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return false;
    }
}
```



解法二：直接抽象成为一维数组，对于一维数组进行二分查找

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int n = matrix.length, m = matrix[0].length;
        int left = 0, right = n * m - 1;
        while (left <= right) {
            int mid = (left + right) / 2;
            int x = mid / m, y = mid % m;
            int t = matrix[x][y];
            if (t == target)
                return true;
            if (t < target)
                left = mid + 1;
            else
                right = mid - 1;
        }
        return false;
    }
}
```



# 寻找峰值

## 题目描述

峰值元素是指其值严格大于左右相邻值的元素。

给你一个整数数组 `nums`，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 **任何一个峰值** 所在位置即可。

你可以假设 `nums[-1] = nums[n] = -∞` 。

你必须实现时间复杂度为 `O(log n)` 的算法来解决此问题。

**示例 1：**

```
输入：nums = [1,2,3,1]
输出：2
解释：3 是峰值元素，你的函数应该返回其索引 2。
```

**示例 2：**

```
输入：nums = [1,2,1,3,5,6,4]
输出：1 或 5 
解释：你的函数可以返回索引 1，其峰值元素为 2；
     或者返回索引 5， 其峰值元素为 6。
```



## 题解

解法一：顺序遍历

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int n = nums.length;
        int[] temp = new int[n + 2];
        for(int i = 0; i < n; i++) {
            temp[i + 1] = nums[i];
        }
        temp[0] = Integer.MIN_VALUE;
        temp[n + 1] = Integer.MIN_VALUE;
        for(int i = 1; i <= n; i++) {
            if(temp[i] > temp[i - 1] && temp[i] > temp[i + 1]) {
                return i - 1;
            }
        }
        return 0;
    }
}
```

解法二：二分查找，当mid对应的数大于mid+1的时候说明峰值在左侧

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int n = nums.length;
        int left = 0;
        int right = n - 1;
        while(left < right) {
            int mid = left + (right - left) / 2;
            if(nums[mid] > nums[mid + 1]) {
                right = mid;
            } else {
                left = mid + 1;
            }
        }
        return left;
    }
}
```

