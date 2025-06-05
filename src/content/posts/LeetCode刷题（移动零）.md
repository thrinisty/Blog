---
title: LeetCode刷题（移动零）
published: 2025-06-05
updated: 2025-06-05
description: '将数组的0移动到数组的末端'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 移动零

7号早上要考英语写作，要写两篇作文，之前在课堂上写的作文评分都不是很高也就是及格边缘，今天和明天需要背一下写作模板，LeetCode刷题部分除了复习以往的题目，就做一道题目练练手保持手感了

## 题目描述

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。

**示例 1:**

```
输入: nums = [0,1,0,3,12]
输出: [1,3,12,0,0]
```

**示例 2:**

```
输入: nums = [0]
输出: [0]
```



## 题解

解法一：忽略原地操作，在list集合中存储非0数，将集合元素复制到数组中，缺少的补0，但是不符合题目要求，其实我还想到用i j 两个下表进行元素互换实现，但是这样就把非零数的顺序打乱了

```java
class Solution {
    public void moveZeroes(int[] nums) {
        List<Integer> list = new ArrayList<>();
        for(Integer num : nums) {
            if(num != 0) {
                list.add(num);
            }
        }
        int n = list.size();
        for(int i = 0; i < n; i++) {
            nums[i] = list.get(i);
        }
        for(int i = n; i < nums.length; i++) {
            nums[i] = 0;
        }
    }
}
```

解法二：原地操作，其实思路一样，确实仔细想想因为有0的存在，覆盖的元素都是被操作过的，直接用数组存储即可，没遍历到的数字不会被覆盖，思路是一样的

```java
class Solution {
    public void moveZeroes(int[] nums) {
        int flag = 0;
        for(int i = 0; i < nums.length; i++) {
            if(nums[i] != 0) {
                nums[flag++] = nums[i]; 
            }
        }
        for(int i = flag; i < nums.length; i++) {
            nums[i] = 0;
        }
    }
}
```

