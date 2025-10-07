---
title: LeetCode刷题（最短无序连续子数组、下一个排列）
published: 2025-10-06
updated: 2025-10-06
description: ''
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 最短无序连续子数组

## 题目描述

给你一个整数数组 `nums` ，你需要找出一个 **连续子数组** ，如果对这个子数组进行升序排序，那么整个数组都会变为升序排序。

请你找出符合题意的 **最短** 子数组，并输出它的长度。

**示例 1：**

```
输入：nums = [2,6,4,8,10,9,15]
输出：5
解释：你只需要对 [6, 4, 8, 10, 9] 进行升序排序，那么整个表都会变为升序排序。
```

**示例 2：**

```
输入：nums = [1,2,3,4]
输出：0
```

**示例 3：**

```
输入：nums = [1]
输出：0
```



## 题解

左往右看：就是一个升序数组，时刻更新max，如果突然是降序，那么它肯定有问题，记录为右边界。

右往左看：就是一个降序数组，时刻更新min，如果突然是升序，那么它肯定有问题，记录为左边界。

![247](../images/247.png)

```java
class Solution {
    public int findUnsortedSubarray(int[] nums) {
        int n = nums.length;
        int min = Integer.MAX_VALUE;
        int max = Integer.MIN_VALUE;
        int left = 0;
        int right = -1;
        for(int i = 0; i < n; i++) {
            if(nums[i] >= max) {
                max = nums[i];
            } else {
                right = i;
            }
            int j = n - i - 1;
            if(nums[j] <= min) {
                min = nums[j];
            } else {
                left = j;
            }
        }
        return right - left + 1;
    }
}
```



# 下一个排列

## 题目描述

整数数组的一个 **排列** 就是将其所有成员以序列或线性顺序排列。

- 例如，`arr = [1,2,3]` ，以下这些都可以视作 `arr` 的排列：`[1,2,3]`、`[1,3,2]`、`[3,1,2]`、`[2,3,1]` 。

整数数组的 **下一个排列** 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 **下一个排列** 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。

- 例如，`arr = [1,2,3]` 的下一个排列是 `[1,3,2]` 。
- 类似地，`arr = [2,3,1]` 的下一个排列是 `[3,1,2]` 。
- 而 `arr = [3,2,1]` 的下一个排列是 `[1,2,3]` ，因为 `[3,2,1]` 不存在一个字典序更大的排列。

给你一个整数数组 `nums` ，找出 `nums` 的下一个排列。

必须**[ 原地 ](https://baike.baidu.com/item/原地算法)**修改，只允许使用额外常数空间。

**示例 1：**

```
输入：nums = [1,2,3]
输出：[1,3,2]
```

**示例 2：**

```
输入：nums = [3,2,1]
输出：[1,2,3]
```

**示例 3：**

```
输入：nums = [1,1,5]
输出：[1,5,1]
```





## 题解

我们希望找到一种方法，能够找到一个大于当前序列的新序列，且变大的幅度尽可能小

我们需要将一个左边的「较小数」与一个右边的「较大数」交换，以能够让当前排列变大，从而得到下一个排列。

同时我们要让这个「较小数」尽量靠右，而「较大数」尽可能小。当交换完成后，「较大数」右边的数需要按照升序重新排列。这样可以在保证新排列大于原来排列的情况下，使变大的幅度尽可能小。

```java
class Solution {
    public void nextPermutation(int[] nums) {
        int n = nums.length;
        int i = n - 2;
        //获取左边的较小数
        while (i >= 0 && nums[i] >= nums[i + 1]) {
            i--;	
        }

        if (i >= 0) {
            //获取最右边的靠右较大数
            int j = n - 1;
            while (j >= 0 && nums[i] >= nums[j]) {
                j--;
            }
            swap(nums, i, j);
        }
        //排序交换后较大数之后的数组部分
        reverse(nums, i + 1, n - 1);

    }

    public void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }

    public void reverse(int[] nums, int start, int end) {
        int i = start;
        int j = end;
        while (i < j) {
            swap(nums, i, j);
            i++;
            j--;
        }
    }
}
```

