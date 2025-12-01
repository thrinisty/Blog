---
title: LeetCode刷题（交错字符串、买卖股票Ⅳ）
published: 2025-11-27
updated: 2025-11-27
description: '交错字符串、买卖股票的最佳时机Ⅳ'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 交错字符串

## 题目描述

给定三个字符串 `s1`、`s2`、`s3`，请你帮忙验证 `s3` 是否是由 `s1` 和 `s2` **交错** 组成的。

两个字符串 `s` 和 `t` **交错** 的定义与过程如下，其中每个字符串都会被分割成若干 **非空** 子字符串：

- `s = s1 + s2 + ... + sn`
- `t = t1 + t2 + ... + tm`
- `|n - m| <= 1`
- **交错** 是 `s1 + t1 + s2 + t2 + s3 + t3 + ...` 或者 `t1 + s1 + t2 + s2 + t3 + s3 + ...`

**注意：**`a + b` 意味着字符串 `a` 和 `b` 连接。

 

**示例 1：**

![479](../images/479.jpg)

```
输入：s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"
输出：true
```

**示例 2：**

```
输入：s1 = "aabcc", s2 = "dbbca", s3 = "aadbbbaccc"
输出：false
```

**示例 3：**

```
输入：s1 = "", s2 = "", s3 = ""
输出：true
```



## 题解

构造dp二维数组，存放的是用前i和j个字符是否可以组成s3字符的i+j个，依次判断i + j的位置是否可以被s1或者s2中的i、j位置防止，当满足一个即可判断对应位置为true

```java
class Solution {
    public boolean isInterleave(String s1, String s2, String s3) {
        int n1 = s1.length();
        int n2 = s2.length();
        int n3 = s3.length();
        if(n1 + n2 != n3) return false;
        boolean[][] dp = new boolean[n1 + 1][n2 + 1];
        dp[0][0] = true;
        s1 = " " + s1;
        s2 = " " + s2;
        s3 = " " + s3;
        for(int i = 0; i <= n1; i++) {
            for(int j = 0; j <= n2; j++) {
                if(i > 0 && s1.charAt(i) == s3.charAt(i + j)) {
                    dp[i][j] = dp[i - 1][j];
                }
                if(j > 0 && s2.charAt(j) == s3.charAt(i + j)) {
                    dp[i][j] = dp[i][j] || dp[i][j - 1];
                }
            }
        }
        return dp[n1][n2];
    }
}
```



# 买卖股票Ⅳ

## 题目描述

给你一个整数数组 `prices` 和一个整数 `k` ，其中 `prices[i]` 是某支给定的股票在第 `i` 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 `k` 笔交易。也就是说，你最多可以买 `k` 次，卖 `k` 次。

**注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1：**

```
输入：k = 2, prices = [2,4,1]
输出：2
解释：在第 1 天 (股票价格 = 2) 的时候买入，在第 2 天 (股票价格 = 4) 的时候卖出，这笔交易所能获得利润 = 4-2 = 2 。
```

**示例 2：**

```
输入：k = 2, prices = [3,2,6,5,0,3]
输出：7
解释：在第 2 天 (股票价格 = 2) 的时候买入，在第 3 天 (股票价格 = 6) 的时候卖出, 这笔交易所能获得利润 = 6-2 = 4 。
     随后，在第 5 天 (股票价格 = 0) 的时候买入，在第 6 天 (股票价格 = 3) 的时候卖出, 这笔交易所能获得利润 = 3-0 = 3 。
```



## 题解

在Ⅲ的基础上纵向的拓展一下

```java
class Solution {
    public int maxProfit(int k, int[] prices) {
        int n = prices.length;
        if(n <= 1) return 0;
        k = Math.min(k, n / 2);
        int[][] buy = new int[k][n];
        int[][] sell = new int[k][n];
        for(int i = 0; i < k; i++) {
            buy[i][0] = -prices[0];
        }
        for(int i = 1; i < n; i++) {
            buy[0][i] = Math.max(buy[0][i - 1], -prices[i]);
            sell[0][i] = Math.max(sell[0][i - 1], buy[0][i - 1] + prices[i]);
            for(int j = 1; j < k; j++) {
                buy[j][i] = Math.max(buy[j][i - 1], sell[j - 1][i - 1] - prices[i]);
                sell[j][i] = Math.max(sell[j][i - 1], buy[j][i - 1] + prices[i]);
            }
        }
        return sell[k - 1][n - 1];
    }
}
```

