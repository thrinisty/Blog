---
title: LeetCode刷题（跳跃游戏）
published: 2025-09-26
updated: 2025-09-26
description: '跳跃游戏、阿里巴巴招聘变体'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 跳跃游戏

## 题目描述

给你一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标，如果可以，返回 `true` ；否则，返回 `false` 。

**示例 1：**

```
输入：nums = [2,3,1,1,4]
输出：true
解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。
```

**示例 2：**

```
输入：nums = [3,2,1,0,4]
输出：false
解释：无论怎样，总会到达下标为 3 的位置。但该下标的最大跳跃长度是 0 ， 所以永远不可能到达最后一个下标。
```



## 题解

贪心算法解决：依次遍历数组的每一项，在当前数组项可达的前提下，用nums[i] + i更行可达的数组项索引，当可达索引超过最后一项的时候可以判断最后一个下表可达。

```java
public class Solution {
    public boolean canJump(int[] nums) {
        int n = nums.length;
        if(n == 1) {
            return true;
        }
        int rightmost = 0;
        for (int i = 0; i < n - 1; ++i) {
            if (i <= rightmost) {
                rightmost = Math.max(rightmost, i + nums[i]);
                if (rightmost >= n - 1) {
                    return true;
                }
            }
        }
        return false;
    }
}
```



这一道题目是阿里巴巴笔试的题目的初级版本，在阿里巴巴的变种题目中题目要求改为了求在可达的前提下求最少的跳跃次数，如果不可达返回-1；

当数组始终可以到达最后一格的时候

```java
class Solution {
    public int jump(int[] nums) {
        int n = nums.length;
        int step = 0;
        int end = 0;
        int maxPosition = 0;
        for(int i = 0; i < n - 1; i++) {
            maxPosition = Math.max(maxPosition, nums[i] + i);
            if(i == end) {
                end = maxPosition;
                step++;
            }
        }
        return step;
    }
}
```



如果数组有可能跳不到最后一格，我们还需要改造算法

### 修改策略

1. **新增条件判断：**
   - **在遍历时检查 `i > rightmost`**
     → 如果当前 `i` 已经超过 `rightmost`，说明无法继续前进，直接返回 `-1`。
   - **在循环结束后检查 `end` 是否足够大**
     → 如果最终的 `end` 无法到达终点，也返回 `-1`。
2. **优化边界：**
   - **初始情况（`n == 1`）**：已经在终点，返回 `0`。

```java
class Solution {
    public int jump(int[] nums) {
        int n = nums.length;
        if (n == 1) return 0; 
        int step = 0;
        int rightmost = 0;
        int end = 0;
        for(int i = 0; i < n - 1; i++) {
            if(rightmost < i) {
                return -1;
            }
            rightmost = Math.max(rightmost, i + nums[i]);
            if(end == i) {
                end = rightmost;
                step++;
            }
            if(end >= n - 1) {
                return step;
            }
        }
        return -1;
    }
}
```

