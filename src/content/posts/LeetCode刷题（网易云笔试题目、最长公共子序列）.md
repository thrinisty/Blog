---
title: LeetCode刷题（网易云笔试题目、最长公共子序列）
published: 2025-10-12
updated: 2025-10-12
description: '仓库租聘收益、最长公共子序列'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 仓库租聘收益

## 题目描述

大概就是股票买卖的升级版本，在基础之上增添了买入卖出的额外支出，还有仓库空闲可以出租时的额外收益



## 题解

增加一个空闲仓库的状态，在初始置为rent，代表当日出租

```java
public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int cent = in.nextInt(); // 每日租金收益
        int pay = in.nextInt();  // 每次买卖的运费
        int n = in.nextInt();    // 天数
        int[] prices = new int[n]; // 每天的货物价格
        
        for (int i = 0; i < n; i++) {
            prices[i] = in.nextInt();
        }
        // DP 状态定义
        int[] buy = new int[n];  // buy[i]: 第i天持有货物时的最大收益
        int[] sell = new int[n]; // sell[i]: 第i天卖出货物时的最大收益
        int[] rent = new int[n]; // rent[i]: 第i天出租仓库时的最大收益
        
        // 初始化
        buy[0] = -prices[0] - pay;   // 第一天买入货物，支付价格和运费
        sell[0] = Integer.MIN_VALUE; // 第一天无法卖出（之前无持有）
        rent[0] = cent;              // 第一天出租仓库
        // DP 状态转移
        for (int i = 1; i < n; i++) {
            // 买入：前一天必须是出租或卖出状态
            buy[i] = Math.max(
                rent[i - 1] - prices[i] - pay,
                sell[i - 1] - prices[i] - pay
            );
            
            // 卖出：前一天必须持有货物
            sell[i] = buy[i - 1] + prices[i] - pay;
            
            // 出租：前一天可以继续出租或从卖出转入
            rent[i] = Math.max(
                rent[i - 1] + cent,
                sell[i - 1] + cent
            );
        }
        // 最终结果是最后一天三种状态的最大值
        int maxProfit = Math.max(rent[n - 1], Math.max(sell[n - 1], buy[n - 1]));
        System.out.println(maxProfit);
    }
```



# 最长公共子序列

## 题目描述

给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长 **公共子序列** 的长度。如果不存在 **公共子序列** ，返回 `0` 。

一个字符串的 **子序列** 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。

- 例如，`"ace"` 是 `"abcde"` 的子序列，但 `"aec"` 不是 `"abcde"` 的子序列。

两个字符串的 **公共子序列** 是这两个字符串所共同拥有的子序列。

**示例 1：**

```
输入：text1 = "abcde", text2 = "ace" 
输出：3  
解释：最长公共子序列是 "ace" ，它的长度为 3 。
```

**示例 2：**

```
输入：text1 = "abc", text2 = "abc"
输出：3
解释：最长公共子序列是 "abc" ，它的长度为 3 。
```

**示例 3：**

```
输入：text1 = "abc", text2 = "def"
输出：0
解释：两个字符串没有公共子序列，返回 0 。
```



## 题解

动态规划：

```
dp[i+1][j+1]表示text1[0,i]范围和text2[0,j]范围构成的最长公共子序列，初始都为0
```

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        int[][] dp = new int[m + 1][n + 1];
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                if(text1.charAt(i) == text2.charAt(j)) {
                    dp[i + 1][j + 1] = dp[i][j] + 1;
                } else {
                    dp[i + 1][j + 1] = Math.max(dp[i][j + 1], dp[i + 1][j]);
                }
            }
        }
        return dp[m][n];
    }
}
```

