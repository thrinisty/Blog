---
title: LeetCode刷题（两数相除、最大的单词长度乘积）
published: 2025-11-30
updated: 2025-11-30
description: '两数相除、最大的单词长度乘积'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 两数相除

## 题目描述

给定两个整数 `a` 和 `b` ，求它们的除法的商 `a/b` ，要求不得使用乘号 `'*'`、除号 `'/'` 以及求余符号 `'%'` 。

 

**注意：**

- 整数除法的结果应当截去（`truncate`）其小数部分，例如：`truncate(8.345) = 8` 以及 `truncate(-2.7335) = -2`
- 假设我们的环境只能存储 32 位有符号整数，其数值范围是 `[−231, 231−1]`。本题中，如果除法结果溢出，则返回 `231 − 1`

 

**示例 1：**

```
输入：a = 15, b = 2
输出：7
解释：15/2 = truncate(7.5) = 7
```

**示例 2：**

```
输入：a = 7, b = -3
输出：-2
解释：7/-3 = truncate(-2.33333..) = -2
```

**示例 3：**

```
输入：a = 0, b = 1
输出：0
```

**示例 4：**

```
输入：a = 1, b = 1
输出：1
```



## 题解

对于特解进行相应处理，因为负数能存的绝对值要大一些，这里会溢出，按题目要求设置为最大值

flag标注数字的最终符号，通过两个ab数来得到，将两个数都改为负数以实现不会溢出（负数的范围大一些），用count计数，当a小于等于b时代表还没有减到最终最接近于0的数，这个时候count代表b的数量，再添加上符号即可

```java
class Solution {
    public int divide(int a, int b) {
        if(b == -1) {
            return a == Integer.MIN_VALUE ? Integer.MAX_VALUE : -a;
        }
        boolean flag = (a <= 0 && b < 0) || (a > 0 && b > 0);
        if(a > 0) a = -a;
        if(b > 0) b = -b;
        int count = 0;
        while(a <= b) {
            a -= b;
            count++;
        }
        return flag ? count : -count;
    }
}
```



# 最大的单词长度乘积

## 题目描述

给定一个字符串数组 `words`，请计算当两个字符串 `words[i]` 和 `words[j]` 不包含相同字符时，它们长度的乘积的最大值。假设字符串中只包含英语的小写字母。如果没有不包含相同字符的一对字符串，返回 0。

**示例 1：**

```
输入：words = ["abcw","baz","foo","bar","fxyz","abcdef"]
输出：16 
解释：这两个单词为 "abcw", "fxyz"。它们不包含相同字符，且长度的乘积最大。
```

**示例 2：**

```
输入：words = ["a","ab","abc","d","cd","bcd","abcd"]
输出：4 
解释：这两个单词为 "ab", "cd"。
```

**示例 3：**

```
输入：words = ["a","aa","aaa","aaaa"]
输出：0 
解释：不存在这样的两个单词。
```



## 题解

依次比较两个字符串是否有相同字符，如果没有则迭代result结果即可

解法一：暴力拆解计算是否有相同的字符

```java
class Solution {
    public int maxProduct(String[] words) {
        int n = words.length;
        int result = 0;
        for(int i = 0; i < n; i++) {
            for(int j = i + 1; j < n; j++) {
                if(!hasSameChar(words[i], words[j])) {
                    result = Math.max(result, words[i].length() * words[j].length());
                }
            }
        }
        return result;
    }

    public boolean hasSameChar(String w1, String w2) {
        for(char c : w1.toCharArray()) {
            if(w2.indexOf(c) != -1) {
                return true;
            }
        }
        return false;
    }
}
```

解法二：更改辅助方法

```java
 public boolean hasSameChar(String w1, String w2) {
        boolean[] count = new boolean[26];
        for(char c : w1.toCharArray()) {
            int index = c - 'a';
            count[index] = true;
        }
        for(char c : w2.toCharArray()) {
            int index = c - 'a';
            if(count[index]) return true;
        }
        return false;
    }
```

