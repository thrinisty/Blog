---
title: LeetCode刷题（盛水最多的容器，数组的第K大元素）
published: 2025-06-12
updated: 2025-06-12
description: '盛水最多的容器'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 盛水最多的容器

## 题目描述

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明：**你不能倾斜容器。

**示例 1：**

![189](../images/189.jpg)

```
输入：[1,8,6,2,5,4,8,3,7]
输出：49 
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
```

**示例 2：**

```
输入：height = [1,1]
输出：1
```

## 题解

和之前接雨水的题目类似，通过两个指针，比较取小的那一个计算出当前结果进行对于结果迭代，直到两个指针重合

```java
class Solution {
    public int maxArea(int[] height) {
        int left = 0;
        int right = height.length - 1;
        int result = 0;
        while(left < right) {
            if(height[left] < height[right]) {
                result = Math.max(result, (right - left) * height[left]);
                left++;
            } else {
                result = Math.max(result, (right - left) * height[right]);
                right--;
            }
        }
        return result;
    }
}
```
