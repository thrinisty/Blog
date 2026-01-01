---
title: LeetCode刷题（分割回文串Ⅱ、最长的斐波那契子序列长度）
published: 2026-01-01
updated: 2026-01-01
description: '分割回文串Ⅱ、最长的斐波那契子序列长度'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 分割回文串Ⅱ

## 题目描述

给定一个字符串 `s`，请将 `s` 分割成一些子串，使每个子串都是回文串。

返回符合要求的 **最少分割次数** 。

**示例 1：**

```
输入：s = "aab"
输出：1
解释：只需一次分割就可将 s 分割成 ["aa","b"] 这样两个回文子串。
```

**示例 2：**

```
输入：s = "a"
输出：0
```

**示例 3：**

```
输入：s = "ab"
输出：1
```



## 题解

先用二维dp构造出回文判断矩阵，再建立结果result数组，代表到当前位置需要的最少分割数，逐个赋予最小分割数

循环的逻辑是如果j为0而0-i为回文就不需要分割赋值为0即可，而如果不为0在0-（j - 1）的最小分割分割即可，并依次迭代最小的分割数

```java
class Solution {
    public int minCut(String s) {
        int n = s.length();
        boolean[][] dp = new boolean[n][n];
        for(int len = 1; len <= n; len++) {
            for(int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                if((s.charAt(i) == s.charAt(j)) && (j - i < 2 || dp[i + 1][j - 1])) {
                    dp[i][j] = true;
                }
            }
        }
        int[] result = new int[n];
        for(int i = 1; i < n; i++) {
            int min = i;
            for(int j = 0; j <= i; j++) {
                if(dp[j][i]) {
                    min = Math.min(min, j == 0 ?  0 : result[j - 1] + 1);
                }
            }
            result[i] = min;
        }
        return result[n - 1];
    }
}
```



# 最长的斐波那契子序列长度

## 题目描述

如果序列 `X_1, X_2, ..., X_n` 满足下列条件，就说它是 *斐波那契式* 的：

- `n >= 3`
- 对于所有 `i + 2 <= n`，都有 `X_i + X_{i+1} = X_{i+2}`

给定一个**严格递增**的正整数数组形成序列 `arr` ，找到 `arr` 中最长的斐波那契式的子序列的长度。如果一个不存在，返回 0 。

*（回想一下，子序列是从原序列 `arr` 中派生出来的，它从 `arr` 中删掉任意数量的元素（也可以不删），而不改变其余元素的顺序。例如， `[3, 5, 8]` 是 `[3, 4, 5, 6, 7, 8]` 的一个子序列）*

**示例 1：**

```
输入: arr = [1,2,3,4,5,6,7,8]
输出: 5
解释: 最长的斐波那契式子序列为 [1,2,3,5,8] 。
```

**示例 2：**

```
输入: arr = [1,3,7,11,12,14,18]
输出: 3
解释: 最长的斐波那契式子序列有 [1,11,12]、[3,11,14] 以及 [7,11,18] 。
```



## 题解

设立辅助的map集合，存放arr数字以及对应下标，设置dp为以i，j结尾的最长长度

```java
class Solution {
    public int lenLongestFibSubseq(int[] arr) {
        Map<Integer, Integer> map = new HashMap<>();
        int n = arr.length;
        for (int i = 0; i < n; i++) {
            map.put(arr[i], i);
        }
        int[][] dp = new int[n][n];
        int result = 0;
        for (int i = 0; i < n; i++) {
            for (int j = i - 1; j >= 0 && arr[j] * 2 > arr[i]; j--) {
                int k = map.getOrDefault(arr[i] - arr[j], -1);
                if (k >= 0) {
                    dp[j][i] = Math.max(dp[k][j] + 1, 3);
                }
                result = Math.max(result, dp[j][i]);
            }
        }
        return result;
    }
}
```

