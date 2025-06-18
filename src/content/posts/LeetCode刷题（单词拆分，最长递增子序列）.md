---
title: LeetCode刷题（单词拆分，最长递增子序列）
published: 2025-06-18
updated: 2025-06-18
description: '单词拆分，最长递增子序列'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 单词拆分

## 题目描述

给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。

**注意：**不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。

**示例 1：**

```
输入: s = "leetcode", wordDict = ["leet", "code"]
输出: true
解释: 返回 true 因为 "leetcode" 可以由 "leet" 和 "code" 拼接成。
```

**示例 2：**

```
输入: s = "applepenapple", wordDict = ["apple", "pen"]
输出: true
解释: 返回 true 因为 "applepenapple" 可以由 "apple" "pen" "apple" 拼接成。
     注意，你可以重复使用字典中的单词。
```

**示例 3：**

```
输入: s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
输出: false
```



## 题解

这一道题我原本是打算用回溯法求解的，思路类似于之前做过的全排列，但是写一半发现如果是后面出现的单词（字典中）出现在结果的最前面，就没有办法求出正常的情况，看了下解析，应该使用动态规划求解，以下是求解思路

设立一个dp数组，下表表示String前n位是否可以用wordDict字典中的单词表示，初始化dp[0]为true

循环遍历字符串，位数从1增至字符串长度，在子循环中判断当前位数的字符串可否由可以组成的前半部分和后半部分包括在字典中的部分拼接形成，如果可以，将对应位数的boolean值设置为true，在循环结束后取dp[s.length()]结果返回

```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        Set<String> set = new HashSet<>(wordDict);
        boolean[] dp = new boolean[s.length() + 1];
        dp[0] = true;
        for(int i = 1; i <= s.length(); i++) {
            for(int j = 0; j < i; j++) {
                if(dp[j] && set.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.length()];
    }
}
```

这里之所以使用HashSet进行存储是得益于Set集合的contains函数时间复杂度是O(1)，不需要像List中需要将是所有的元素遍历后才可以得到最终的结果

很好的题目，巩固了HashSet部分的知识，顺便复习了一下字符串的相关函数substring



# 最长的递增子序列

## 题目描述

给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。

**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。

 

**示例 1：**

```
输入：nums = [10,9,2,5,3,7,101,18]
输出：4
解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。
```

**示例 2：**

```
输入：nums = [0,1,0,3,2,3]
输出：4
```

**示例 3：**

```
输入：nums = [7,7,7,7,7,7,7]
输出：1
```



## 题解

运用动态规划求解，对于结果dp而言，每单个数字，都是长度为1的递增子序列，用1将所有的dp赋值，从起始位置开始循环遍历，子循环变量j为包含j的最长子序列的结果，当其对应的数字小于外部循环对应的数字时，用dp[j] + 1迭代dp[i]的结果

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        if(nums.length == 0) {
            return 0;
        }
        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < i; j++) {
                if(nums[j] < nums[i]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
        }
        int result = 0;
        for(int num : dp) {
            result = Math.max(result, num);
        }
        return result;
    }
}
```

或者也可以在for循环中对于结果进行迭代，速度会更快一点

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        if(nums.length == 0) {
            return 0;
        }
        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        int result = 0;
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < i; j++) {
                if(nums[j] < nums[i]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            result = Math.max(result, dp[i]);
        }
        return result;
    }
}
```

