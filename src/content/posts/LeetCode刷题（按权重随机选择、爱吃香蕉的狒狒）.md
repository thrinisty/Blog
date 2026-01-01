---
title: LeetCode刷题（按权重随机选择、爱吃香蕉的狒狒）
published: 2025-12-22
updated: 2025-12-22
description: '按权重随机选择、爱吃香蕉的狒狒'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 按权重随机选择

## 题目描述

给定一个正整数数组 `w` ，其中 `w[i]` 代表下标 `i` 的权重（下标从 `0` 开始），请写一个函数 `pickIndex` ，它可以随机地获取下标 `i`，选取下标 `i` 的概率与 `w[i]` 成正比

例如，对于 `w = [1, 3]`，挑选下标 `0` 的概率为 `1 / (1 + 3) = 0.25` （即，25%），而选取下标 `1` 的概率为 `3 / (1 + 3) = 0.75`（即，75%）。

也就是说，选取下标 `i` 的概率为 `w[i] / sum(w)` 。

**示例 1：**

```
输入：
inputs = ["Solution","pickIndex"]
inputs = [[[1]],[]]
输出：
[null,0]
解释：
Solution solution = new Solution([1]);
solution.pickIndex(); // 返回 0，因为数组中只有一个元素，所以唯一的选择是返回下标 0。
```

**示例 2：**

```
输入：
inputs = ["Solution","pickIndex","pickIndex","pickIndex","pickIndex","pickIndex"]
inputs = [[[1,3]],[],[],[],[],[]]
输出：
[null,1,1,1,1,0]
解释：
Solution solution = new Solution([1, 3]);
solution.pickIndex(); // 返回 1，返回下标 1，返回该下标概率为 3/4 。
solution.pickIndex(); // 返回 1
solution.pickIndex(); // 返回 1
solution.pickIndex(); // 返回 1
solution.pickIndex(); // 返回 0，返回下标 0，返回该下标概率为 1/4 。

由于这是一个随机问题，允许多个答案，因此下列输出都可以被认为是正确的:
[null,1,1,1,1,0]
[null,1,1,1,1,1]
[null,1,1,1,0,0]
[null,1,1,1,0,1]
[null,1,0,1,0,0]
......
诸若此类。
```



## 题解

二分查找+前缀和，构造前缀和后，运用二分查找差对应的元素

```java
class Solution {
    int[] pre;
    public Solution(int[] w) {
        pre = new int[w.length];
        pre[0] = w[0];
        for(int i = 1; i < w.length; i++) {
            pre[i] = pre[i - 1] + w[i];
        }
    }
    
    public int pickIndex() {
        int x = (int) (Math.random() * pre[pre.length - 1]) + 1;
        return binarySearch(x);
    }

    public int binarySearch(int x) {
        int left = 0;
        int right = pre.length - 1;
        while(left <= right) {
            int mid = left + (right - left) / 2;
            if(pre[mid] == x) {
                return mid;
            } else if(x < pre[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return left;
    }
}
```



# 爱吃香蕉的狒狒

## 题目描述

狒狒喜欢吃香蕉。这里有 `N` 堆香蕉，第 `i` 堆中有 `piles[i]` 根香蕉。警卫已经离开了，将在 `H` 小时后回来。

狒狒可以决定她吃香蕉的速度 `K` （单位：根/小时）。每个小时，她将会选择一堆香蕉，从中吃掉 `K` 根。如果这堆香蕉少于 `K` 根，她将吃掉这堆的所有香蕉，然后这一小时内不会再吃更多的香蕉，下一个小时才会开始吃另一堆的香蕉。 

狒狒喜欢慢慢吃，但仍然想在警卫回来前吃掉所有的香蕉。

返回她可以在 `H` 小时内吃掉所有香蕉的最小速度 `K`（`K` 为整数）。

**示例 1：**

```
输入: piles = [3,6,7,11], H = 8
输出: 4
```

**示例 2：**

```
输入: piles = [30,11,23,4,20], H = 5
输出: 30
```

**示例 3：**

```
输入: piles = [30,11,23,4,20], H = 6
输出: 23
```



## 题解

二分查找找符合条件的结果，边界最小为1，最大为香蕉最大数（此时需要耗费的时间最小），其中值得注意的是向上取整的部分的表达

```java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int max = Integer.MIN_VALUE;
        for(int pile : piles) {
            max = Math.max(max, pile);
        }
        int left = 1;
        int right = max;
        while(left <= right) {
            int mid = left + (right - left) / 2;
            if(getHour(mid, piles) > h) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        return left;
    }

    private int getHour(int speed, int[] piles){
        int hour = 0;
        for(int pile : piles) {
            //向上取整
            hour += (pile - 1) / speed + 1;
        }
        return hour;
    }
}
```

