---
title: LeetCode刷题（删除有序数组中的重复项、轮转数组）
published: 2025-10-23
updated: 2025-10-23
description: '删除有序数组中的重复项、允许重复变种、轮转数组'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 删除有序数组中的重复项

## 题目描述

给你一个 **非严格递增排列** 的数组 `nums` ，请你**[ 原地](http://baike.baidu.com/item/原地算法)** 删除重复出现的元素，使每个元素 **只出现一次** ，返回删除后数组的新长度。元素的 **相对顺序** 应该保持 **一致** 。然后返回 `nums` 中唯一元素的个数。

考虑 `nums` 的唯一元素的数量为 `k`。去重后，返回唯一元素的数量 `k`。

`nums` 的前 `k` 个元素应包含 **排序后** 的唯一数字。下标 `k - 1` 之后的剩余元素可以忽略。

**示例 1：**

```
输入：nums = [1,1,2]
输出：2, nums = [1,2,_]
解释：函数应该返回新的长度 2 ，并且原数组 nums 的前两个元素被修改为 1, 2 。不需要考虑数组中超出新长度后面的元素。
```

**示例 2：**

```
输入：nums = [0,0,1,1,1,2,2,3,3,4]
输出：5, nums = [0,1,2,3,4,_,_,_,_,_]
解释：函数应该返回新的长度 5 ， 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4 。不需要考虑数组中超出新长度后面的元素。
```



## 题解

设置i与j，其中i用于指向即将插入后面不重复元素的位置，用j进行数组的遍历，从1开始，如果遍历到的元素和上一个插入的元素不同，就将元素插入i位置，并向后移动i，更新

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int n = nums.length;
        int i = 1;
        for(int j = 1; j < n; j++) {
            if(nums[i - 1] != nums[j]) {
                nums[i] = nums[j];
                i++;
            }
        }
        return i;
    }
}
```



# 删除有序数组中的重复项Ⅱ

## 题目描述

给你一个有序数组 `nums` ，请你**[ 原地](http://baike.baidu.com/item/原地算法)** 删除重复出现的元素，使得出现次数超过两次的元素**只出现两次** ，返回删除后数组的新长度。

不要使用额外的数组空间，你必须在 **[原地 ](https://baike.baidu.com/item/原地算法)修改输入数组** 并在使用 O(1) 额外空间的条件下完成。

**示例 1：**

```
输入：nums = [1,1,1,2,2,3]
输出：5, nums = [1,1,2,2,3]
解释：函数应返回新长度 length = 5, 并且原数组的前五个元素被修改为 1, 1, 2, 2, 3。 不需要考虑数组中超出新长度后面的元素。
```

**示例 2：**

```
输入：nums = [0,0,1,1,1,1,2,3,3]
输出：7, nums = [0,0,1,1,2,3,3]
解释：函数应返回新长度 length = 7, 并且原数组的前七个元素被修改为 0, 0, 1, 1, 2, 3, 3。不需要考虑数组中超出新长度后面的元素。
```



## 题解

自己手写的有些复杂，其中flag代表是否可以插入相同的元素

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int i = 1;
        boolean flag = true;
        int num = nums[0];
        int n = nums.length;
        for(int j = 1; j < n; j++) {
            if(flag) {
                nums[i++] = nums[j];
                if(num != nums[j]) {
                    num = nums[j];
                    flag = true;
                } else {
                    flag = false;
                }
            } else {
                if(num != nums[j]) {
                    nums[i++] = nums[j];
                    num = nums[j];
                    flag = true;
                }
            }
        }
        return i;
    }
}
```

精简一下，并可拓展，例如允许3个4个

当元素重复的时候就count++，如果元素重复数量大于2则不加入

这里没有处理nums长度为0的情况，如果为0或者空，返回0；

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int i = 1;
        int n = nums.length;
        int count = 1;
        for(int j = 1; j < n; j++) {
            if(nums[i - 1] == nums[j]) {
                count++;
                if(count <= 2) {
                    nums[i++] = nums[j];
                }
            } else {
                nums[i++] = nums[j];
                count = 1;
            }
        }
        return i;
    }
}
```



# 轮转数组

## 题目描述

给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

**示例 1:**

```
输入: nums = [1,2,3,4,5,6,7], k = 3
输出: [5,6,7,1,2,3,4]
解释:
向右轮转 1 步: [7,1,2,3,4,5,6]
向右轮转 2 步: [6,7,1,2,3,4,5]
向右轮转 3 步: [5,6,7,1,2,3,4]
```

**示例 2:**

```
输入：nums = [-1,-100,3,99], k = 2
输出：[3,99,-1,-100]
解释: 
向右轮转 1 步: [99,-1,-100,3]
向右轮转 2 步: [3,99,-1,-100]
```



## 题解

解法一：建立辅助数组，通过取模的方式进行移动

```java
class Solution {
    public void rotate(int[] nums, int k) {
        int n = nums.length;
        int[] result = new int[n];
        for(int i = 0; i < n; i++) {
            result[(i + k) % n] = nums[i];
        }
        for(int i = 0; i < n; i++) {
            nums[i] = result[i];
        }
    }
}
```

