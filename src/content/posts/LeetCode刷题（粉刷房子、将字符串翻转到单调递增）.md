---
title: LeetCode刷题（粉刷房子、将字符串翻转到单调递增）
published: 2025-12-30
updated: 2025-12-30
description: '粉刷房子、将字符串翻转到单调递增'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 粉刷房子

## 题目描述

假如有一排房子，共 `n` 个，每个房子可以被粉刷成红色、蓝色或者绿色这三种颜色中的一种，你需要粉刷所有的房子并且使其相邻的两个房子颜色不能相同。

当然，因为市场上不同颜色油漆的价格不同，所以房子粉刷成不同颜色的花费成本也是不同的。每个房子粉刷成不同颜色的花费是以一个 `n x 3` 的正整数矩阵 `costs` 来表示的。

例如，`costs[0][0]` 表示第 0 号房子粉刷成红色的成本花费；`costs[1][2]` 表示第 1 号房子粉刷成绿色的花费，以此类推。

请计算出粉刷完所有房子最少的花费成本。

**示例 1：**

```
输入: costs = [[17,2,17],[16,16,5],[14,3,19]]
输出: 10
解释: 将 0 号房子粉刷成蓝色，1 号房子粉刷成绿色，2 号房子粉刷成蓝色。
     最少花费: 2 + 5 + 3 = 10。
```

**示例 2：**

```
输入: costs = [[7,6,2]]
输出: 2
```

 

## 题解

建立三个状态，依次在不同的颜色情况下进行转移即可

```java
class Solution {
    public int minCost(int[][] costs) {
        int n = costs.length;
        int a = costs[0][0];
        int b = costs[0][1];
        int c = costs[0][2];
        for(int i = 1; i < n; i++) {
            int ap = Math.min(b, c) + costs[i][0];
            int bp = Math.min(a, c) + costs[i][1];
            int cp = Math.min(a, b) + costs[i][2];
            a = ap; b = bp; c = cp;
        }
        return Math.min(a, Math.min(b, c));
    }
}
```



# 将字符串翻转到单调递增

## 题目描述

如果一个由 `'0'` 和 `'1'` 组成的字符串，是以一些 `'0'`（可能没有 `'0'`）后面跟着一些 `'1'`（也可能没有 `'1'`）的形式组成的，那么该字符串是 **单调递增** 的。

我们给出一个由字符 `'0'` 和 `'1'` 组成的字符串 s，我们可以将任何 `'0'` 翻转为 `'1'` 或者将 `'1'` 翻转为 `'0'`。

返回使 s **单调递增** 的最小翻转次数。

**示例 1：**

```
输入：s = "00110"
输出：1
解释：我们翻转最后一位得到 00111.
```

**示例 2：**

```
输入：s = "010110"
输出：2
解释：我们翻转得到 011111，或者是 000111。
```

**示例 3：**

```
输入：s = "00011000"
输出：2
解释：我们翻转得到 00000000。
```



## 题解

设立两个数组保存当目前为0/1的最小改变的次数

当前元素为如果为0，当前防止为0的结果则从前一位0直接迁移即可（0后跟着0），而如果要为1，则要选取前一位0/1中较小的哪一个再+1

反之如果当前元素为1，则如果要使防止0需要从0的结果+1，要使放置1，则从0或者1的前一位迁过来即可无需改变

```java
class Solution {
    public int minFlipsMonoIncr(String s) {
        int n = s.length();
        int[] result0 = new int[n];
        int[] result1 = new int[n];
        if(s.charAt(0) == '1') {
            result0[0] = 1;
        } else {
            result1[0] = 1;
        }

        for(int i = 1; i < n; i++) {
            char c = s.charAt(i);
            if(c == '0') {
                result0[i] = result0[i - 1];
                result1[i] = Math.min(result1[i - 1], result0[i - 1]) + 1; 
            } else {
                result0[i] = result0[i - 1] + 1;
                result1[i] = Math.min(result0[i - 1], result1[i - 1]);
            }
        }
        return Math.min(result0[n - 1], result1[n - 1]);
    }
}
```

