---
title: LeetCode刷题（数组第K大元素）
published: 2025-06-13
updated: 2025-06-13
description: '数组第K大元素'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 数组的第K大元素

## 题目描述

给定整数数组 `nums` 和整数 `k`，请返回数组中第 `**k**` 个最大的元素。

请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。

你必须设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

**示例 1:**

```
输入: [3,2,1,5,6,4], k = 2
输出: 5
```

**示例 2:**

```
输入: [3,2,3,1,2,4,5,5,6], k = 4
输出: 4
```



## 题解

我们先来回顾一下数组的快速排序，我们的目的是在其基础上进行修改

```java
class Solution {
    public int[] sortArray(int[] nums) {
        quickSort(nums, 0, nums.length - 1);
        return nums;
    }

    public void quickSort(int[] nums, int left, int right) {
        if(left >= right) {
            return;
        }
        int mid = partition(nums, left, right); 
        quickSort(nums, left, mid - 1);
        quickSort(nums, mid + 1, right);
    }

    public int partition(int[] nums, int left, int right) {
        int midNum = nums[left];
        int i = left + 1;
        int j = right;
        while(i <= j) {
            while(i <= j && nums[i] < midNum) {
                i++;
            }
            while(i <= j && nums[j] > midNum) {
                j--;
            }
            if(i <= j) {
                swap(nums, i, j);
                i++;
                j--;
            }
        }
        swap(nums, j, left);
        return j;
    }

    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

我们的partition方法和swap方法都可以复用，只需要修改Quick功能函数的逻辑即可，在快速排序中我们需要对于所选择中间值的左右两侧同时进行递归排序

而在快速选择排序中我们可以先对于得到的所选值进行判断，如果是选择值的索引位置是我们需要的目标索引直接返回即可，因为左右两侧虽然没有排序完毕，但因为左边的值都是小于选取目标，右边同理，选择值的索引位置与排序过后的位置相同

索引不一致的时候，递归选择一侧重新调用partition选取值即可

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        int target = nums.length - k;
        return quickSelect(nums, 0, nums.length - 1, target);
    }

    public int quickSelect(int[] nums, int left, int right, int target) {
        if(left == right) {
            return nums[left];
        }
        int mid = partition(nums, left, right);
        if(mid == target) {
            return nums[mid];
        } else if(mid < target) {
            return quickSelect(nums, mid + 1, right, target);
        } else {
            return quickSelect(nums, left, mid - 1, target);
        }
    }

    public int partition(int[] nums, int left, int right) {
        int midNum = nums[left];
        int i = left + 1;
        int j = right;
        while(i <= j) {
            while(i <= j && nums[i] < midNum) {
                i++;
            }
            while(i <= j && nums[j] > midNum) {
                j--;
            }
            if(i <= j) {
                swap(nums, i, j);
                i++;
                j--;
            }
        }
        swap(nums, j, left);
        return j;
    }

    public void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

