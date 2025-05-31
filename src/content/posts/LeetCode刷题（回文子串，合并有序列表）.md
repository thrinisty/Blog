---
title: LeetCode刷题（回文子串，合并有序列表）
published: 2025-05-31
updated: 2025-05-31
description: '回文子串，合并两个有序列表'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 回文子串

## 题目描述

给你一个字符串 `s` ，请你统计并返回这个字符串中 **回文子串** 的数目。

**回文字符串** 是正着读和倒过来读一样的字符串。

**子字符串** 是字符串中的由连续字符组成的一个序列。

 

**示例 1：**

```
输入：s = "abc"
输出：3
解释：三个回文子串: "a", "b", "c"
```

**示例 2：**

```
输入：s = "aaa"
输出：6
解释：6个回文子串: "a", "a", "a", "aa", "aa", "aaa"
```

 

## 题解

动态规划：

```
状态转移方程：当 s[i] == s[j] && (j - i < 2 || dp[i + 1][j - 1]) 时，dp[i][j]=true，否则为false
```

判断两个字符间的内容是否为回文串：先看左右两个字符是否相同，不同则不为回文，相同则分为三种情况

1.字符串长为1，即 j - i == 0，单个字符也算回文

2.字符串长为2，即 j - i == 1，xx算回文串，且中间没有内容

3.其余情况，中间内容为回文串，两边字符相同，也是回文子串



我在第一次写的时候是这样写的：自认为可以完全遍历所有子串，但是遍历的方式出了点问题，顺序是 `i` 从 `0` 到 `n-1`，`j` 从 `i` 到 `n-1`，这意味着在计算 `dp[i][j]` 时，`dp[i + 1][j - 1]` 可能还没有被计算。

```java
class Solution {
    public int countSubstrings(String s) {
        int n = s.length();
        boolean[][] dp = new boolean[n][n];
        int result = 0;
        for(int i = 0; i < n; i++) {
            for(int j = i; j < n; j++) {
                if(s.charAt(i) == s.charAt(j) && (j - i == 0 || j - i == 1 || dp[i + 1][j - 1])) {
                    dp[i][j] = true;
                    result++;
                } else {
                    dp[i][j] = false;
                }
            }
        }
        return result;
    }
}
```

正确解法：

```java
class Solution {
    public int countSubstrings(String s) {
        int n = s.length();
        boolean[][] dp = new boolean[n][n];
        int result = 0;
        for(int j = 0; j < n; j++) {
            for(int i = 0; i <= j; i++) {
                if(s.charAt(i) == s.charAt(j) && (j - i == 0 || j - i == 1 || dp[i + 1][j - 1])) {
                    dp[i][j] = true;
                    result++;
                } 
            }
        }
        return result;
    }
}
```

或者我们使用更加便于理解的方式（我更倾向于这种方式）

以下的解法类似于之前做过的戳气球，按照长度递增，这样就可以保证，长度长的依赖的长度小的回文被优先判断出来。

```java
class Solution {
    public int countSubstrings(String s) {
        int n = s.length();
        boolean[][] dp = new boolean[n][n];
        int result = 0;
        for(int len = 1; len <= n; len++) {
            for(int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                if(s.charAt(i) == s.charAt(j) && (j - i == 0 || j - i == 1 || dp[i + 1][j - 1])) {
                    dp[i][j] = true;
                    result++;
                } 
            }
        }
        return result;
    }
}
```



# 合并有序列表

## 题目描述

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

**示例 1：**

![171](../images/171.jpg)

```
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]
```

**示例 2：**

```
输入：l1 = [], l2 = []
输出：[]
```

**示例 3：**

```
输入：l1 = [], l2 = [0]
输出：[0]
```



## 题解

很久没有做链表的题了，有点乱不清楚指针，看了答案思路就清晰多了

```java
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode result = new ListNode();
        ListNode p = result;
        while(list1 != null && list2 != null) {
            if(list1.val <= list2.val) {
                p.next = list1;
                list1 = list1.next;
                p = p.next;
            } else{
                p.next = list2;
                list2 = list2.next;
                p = p.next;
            }
        }
        p.next = list1 == null ? list2 : list1; 
        return result.next;
    }
}
```

