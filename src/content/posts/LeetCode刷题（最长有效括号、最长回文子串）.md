---
title: LeetCode刷题（最长有效括号、最长回文子串）
published: 2025-10-08
updated: 2025-10-08
description: ''
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 最长有效括号

## 题目描述

给你一个只包含 `'('` 和 `')'` 的字符串，找出最长有效（格式正确且连续）括号 子串 的长度。

左右括号匹配，即每个左括号都有对应的右括号将其闭合的字符串是格式正确的，比如 `"(()())"`。

**示例 1：**

```
输入：s = "(()"
输出：2
解释：最长有效括号子串是 "()"
```

**示例 2：**

```
输入：s = ")()())"
输出：4
解释：最长有效括号子串是 "()()"
```

**示例 3：**

```
输入：s = ""
输出：0
```



## 题解

维护一个栈结构，初始化放入一个-1代表-1下表是初始最后一个不匹配的位置，遍历字符串，如果当前元素为（，则将其下表压入栈中，如果不是则先弹出一个元素，再分为两种情况讨论

第一种：如果为空，代表这个）是没有办法匹配的我们需要合法情况的，我们将其位置入栈代表这是目前遍历顺序中最后一个不匹配的位置，

第二种：如果不为空，我们用i-stack.peek()代表目前构造出来从最后一个不匹配位置开始的合法字符串的长度大小，并迭代result结果

```java
class Solution {
    public int longestValidParentheses(String s) {
        int result = 0;
        Stack<Integer> stack = new Stack<>();
        stack.push(-1);
        for(int i = 0; i < s.length(); i++) {
            if(s.charAt(i) == '(') {
                stack.push(i);	
                //这里如果是push出来的是（代表的是拿出一个（和）匹配
                //如果是push的是），代表拿出来的是原先的不匹配最大）放的位置，在接下来我们会去更新它为新的不匹配最大）放的位置
            } else {
                stack.pop();
                if(stack.isEmpty()) {	
                    stack.push(i);
                } else {
                    result = Math.max(result, i - stack.peek());
                }
            }
        }
        return result;
    }
}
```



# 最长回文子串

## 题目描述

给你一个字符串 `s`，找到 `s` 中最长的 回文 子串。

**示例 1：*

```
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。
```

**示例 2：**

```
输入：s = "cbbd"
输出："bb"
```



## 题解

解法一：动态规划

和之前求解回文子串数量的方式类似，通过dp二维数组去判断是否是一个回文子串，在可能的情况去更新result字符串结果

重复substring对性能有些影响，我们使用记录start与end位置的方式进行优化

```java
class Solution {
    public String longestPalindrome(String s) {
        int n = s.length();
        boolean[][] dp = new boolean[n][n];
        int start = 0;
        int end = 0;
        for(int len = 1; len <= n; len++) {
            for(int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                if(s.charAt(i) == s.charAt(j) && (j - i == 0 || j - i == 1 || dp[i + 1][j - 1])) {
                    dp[i][j] = true;
                    start = i;
                    end = j;
                } 
            }
        }
        return s.substring(start, end + 1);
    }
}
```



解法二：中心扩散法

选取每个元素向左右两侧扩散，当大小大于原先的时候迭代

```java
class Solution {
    public String longestPalindrome(String s) {
        int length = s.length();
        int maxStart = 0;
        int maxLength = 0;
        for (int i = 0; i < length; i++) {
            // j=0表示中心节点只有 i，j=1，表示中心节点有两个 i，i+1;
            for (int j = 0; j <= 1; j++) {
                int l = i;
                int r = i + j;

                while (l >= 0 && r < length && s.charAt(l) == s.charAt(r)) {
                    l--;
                    r++;
                }

                // 回溯到回文字符串的起始和结束位置
                l++;
                r--;

                // 比较并保存最长的字符串起始位置和长度。
                if (maxLength < r - l + 1) {
                    maxLength = r - l + 1;
                    maxStart = l;
                }
            }
        }
        return s.substring(maxStart, maxStart + maxLength);
    }
}
```

