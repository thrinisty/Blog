---
title: LeetCode刷题（每日温度）
published: 2025-05-30
updated: 2025-05-30
description: '每日温度（非暴力）'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 每日温度

## 题目描述

给定一个整数数组 `temperatures` ，表示每天的温度，返回一个数组 `answer` ，其中 `answer[i]` 是指对于第 `i` 天，下一个更高温度出现在几天后。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

**示例 1:**

```
输入: temperatures = [73,74,75,71,69,72,76,73]
输出: [1,1,4,2,1,1,0,0]
```

**示例 2:**

```
输入: temperatures = [30,40,50,60]
输出: [1,1,1,0]
```

**示例 3:**

```
输入: temperatures = [30,60,90]
输出: [1,1,0]
```



## 题解

方法一：暴力拆解 自己想出来的一般都超时 :(  

两重for循环，依次遍历后续直到遇到更高的温度，返回下表差之后break跳出循环

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] result = new int[n];
        for(int i = 0; i < n; i++) {
            for(int j = i; j < n; j++) {
                if(temperatures[i] < temperatures[j]) {
                    result[i] = j - i;
                    break;
                }
            }
        }
        return result;
    }
}
```

测试用例是可以通过的，但是提交的时候超时，测试用例中有一个非常夸张的样例，O(n^2)直接爆炸了，得使用时间复杂度更小的解法完成



方法二：

从反方向开始遍历

在上一种解法中，起始在第二层for循环的时候重复计算了很多没必要的情况，例如 

```
33 21 20 34
3  2  1  0
```

我们在判断33后续的更高温度的时候，先看21是否高于33，如果不高于33，可以不用依次遍历21后续的数字，而是直接跳转到21对应result中的下表（因为直到该下表前都没有大于21的温度，重复遍历没有必要），当21结果为0的时候，后续也没有温度更高的结果，直接使对应结果置为0     

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] result = new int[n];
        for(int i = n - 2; i >= 0; i--) {
            for(int j = i + 1; j < n; j += result[j]) {
                if(temperatures[i] < temperatures[j]) {
                    result[i] = j - i;
                    break;
                } else if(result[j] == 0) {
                    result[i] = 0;
                    break;
                } else {
                    continue;
                }
            }
        }
        return result;
    }
}
```



# 不同路径

## 题目描述

一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

 

**示例 1：**

![170](../images/170.png)

```
输入：m = 3, n = 7
输出：28
```

**示例 2：**

```
输入：m = 3, n = 2
输出：3
解释：
从左上角开始，总共有 3 条路径可以到达右下角。
1. 向右 -> 向下 -> 向下
2. 向下 -> 向下 -> 向右
3. 向下 -> 向右 -> 向下
```

**示例 3：**

```
输入：m = 7, n = 3
输出：28
```

**示例 4：**

```
输入：m = 3, n = 3
输出：6
```



## 题解

动态规划：和之前的最小路径类似，这里循环的操作是将上节点和左节点的走法数相加

```java
class Solution {
    public int uniquePaths(int m, int n) {
        int[][] result = new int[m][n];
        for(int i = 0; i < m; i++) {
            result[i][0] = 1;
        }
        for(int j = 0; j < n; j++) {
            result[0][j] = 1;
        }
        for(int i = 1; i < m; i++) {
            for(int j = 1; j < n; j++) {
                result[i][j] = result[i - 1][j] + result[i][j - 1];
            }
        }
        return result[m - 1][n - 1];
    }
}
```



# 完全平方数

## 题目表述

给你一个整数 `n` ，返回 *和为 `n` 的完全平方数的最少数量* 。

**完全平方数** 是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，`1`、`4`、`9` 和 `16` 都是完全平方数，而 `3` 和 `11` 不是。

 

**示例 1：**

```
输入：n = 12
输出：3 
解释：12 = 4 + 4 + 4
```

**示例 2：**

```
输入：n = 13
输出：2
解释：13 = 4 + 9
```

 

## 题解

运用一个数组存储对应的数字可以被完全平方数表示的最小平方数个数，第二层for循环表示，每一个直到 j * j 大于 i 的情况，而min结果是剩余的数的最小平方数个数，迭代min找出，个数最小的情况，最后再加上这个i对应的完全平方数（result = min + 1）

```java
class Solution {
    public int numSquares(int n) {
        int[] result = new int [n + 1];
        for(int i = 1; i <= n; i++) {
            int min = Integer.MAX_VALUE;
            for(int j = 1; j * j <= i; j++) {
                min = Math.min(min, result[i - j * j]);
            }
            result[i] = min + 1;
        }
        return result[n];
    }
}
```

