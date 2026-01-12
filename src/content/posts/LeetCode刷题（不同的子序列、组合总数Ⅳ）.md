---
title: LeetCode刷题（不同的子序列、组合总数Ⅳ）
published: 2026-01-02
updated: 2026-01-02
description: '不同的子序列、组合总数Ⅳ'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 不同的子序列

## 题目描述

给定一个字符串 `s` 和一个字符串 `t` ，计算在 `s` 的子序列中 `t` 出现的个数。

字符串的一个 **子序列** 是指，通过删除一些（也可以不删除）字符且不干扰剩余字符相对位置所组成的新字符串。（例如，`"ACE"` 是 `"ABCDE"` 的一个子序列，而 `"AEC"` 不是）

题目数据保证答案符合 32 位带符号整数范围。

**示例 1：**

```
输入：s = "rabbbit", t = "rabbit"
输出：3
解释：
如下图所示, 有 3 种可以从 s 中得到 "rabbit" 的方案。
rabbbit
rabbbit
rabbbit
```

**示例 2：**

```
输入：s = "babgbag", t = "bag"
输出：5
解释：
如下图所示, 有 5 种可以从 s 中得到 "bag" 的方案。 
babgbag
babgbag
babgbag
babgbag
babgbag
```



## 题解

dp表示到第i个地方有到j地方的子序列数，初始化s有空字符串子序列都为1种可能

如果字符串字符匹配，则子序列数为同时取i j 的子序列数加上不取当前字符对应的子序列数

如果不匹配则子序列数为不取i的子序列数

```java
class Solution {
    public int numDistinct(String s, String t) {
        int m = s.length(), n = t.length();
        if (m < n) return 0;
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 0; i <= m; i++) {
            dp[i][0] = 1;
        }
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (s.charAt(i) == t.charAt(j)) {
                    dp[i + 1][j + 1] = dp[i][j] + dp[i][j + 1];
                } else {
                    dp[i + 1][j + 1] = dp[i][j + 1];
                }
            }
        }
        
        return dp[m][n];
    }
}
```



# 组合总数Ⅳ

## 题目描述

给定一个由 **不同** 正整数组成的数组 `nums` ，和一个目标整数 `target` 。请从 `nums` 中找出并返回总和为 `target` 的元素组合的个数。数组中的数字可以在一次排列中出现任意次，但是顺序不同的序列被视作不同的组合。

题目数据保证答案符合 32 位整数范围。

 

**示例 1：**

```
输入：nums = [1,2,3], target = 4
输出：7
解释：
所有可能的组合为：
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)
请注意，顺序不同的序列被视作不同的组合。
```

**示例 2：**

```
输入：nums = [9], target = 3
输出：0
```



## 题解

利用动态规划求解，依次对每一个target值得总和数进行计算对每一种num值看在等于的情况加上当前的总和数

:::note

注意这里和完全背包做区分，完全背包得到的排列数是不重复的，即122和221等价，其求的是组合数

:::

```java
class Solution {
    public int combinationSum4(int[] nums, int target) {
        int[] dp = new int[target + 1];
        dp[0] = 1;
        for(int i = 1; i <= target; i++) {
            for(int num : nums) {
                if(i - num >= 0) {
                    dp[i] += dp[i - num];
                }
            }
        }
        return dp[target];
    }
}
```

