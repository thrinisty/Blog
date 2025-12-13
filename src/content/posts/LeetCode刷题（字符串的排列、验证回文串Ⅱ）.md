---
title: LeetCode刷题（字符串的排列、验证回文串Ⅱ）
published: 2025-12-05
updated: 2025-12-05
description: '字符串的排列、验证回文串Ⅱ'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 字符串的排列

## 题目描述

给定两个字符串 `s1` 和 `s2`，写一个函数来判断 `s2` 是否包含 `s1` 的某个变位词。

换句话说，第一个字符串的排列之一是第二个字符串的 **子串** 。

**示例 1：**

```
输入: s1 = "ab" s2 = "eidbaooo"
输出: True
解释: s2 包含 s1 的排列之一 ("ba").
```

**示例 2：**

```
输入: s1= "ab" s2 = "eidboaoo"
输出: False
```

 

**提示：**

- `1 <= s1.length, s2.length <= 104`
- `s1` 和 `s2` 仅包含小写字母



## 题解

解法一：运用滑动窗口依次判断每个窗口是否是异位词，判断的方式是截取字符串排序

这样的瓶颈是每次判断都需要排序字符串

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        char[] str = s1.toCharArray();
        Arrays.sort(str);
        String target = new String(str);
        int m = str.length;
        int n = s2.length();
        for(int i = 0; i < n - m + 1; i++) {
            String current = s2.substring(i, i + m);
            char[] temp = current.toCharArray();
            Arrays.sort(temp);
            current = new String(temp);
            if(target.equals(current)) {
                return true;
            }
        }
        return false;
    }
}
```

解法二：还是利用滑动窗口，不过采用26位的数组记录各个字符数量，数量保持一致的时候证明是异位词

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        if (m > n) {
            return false;
        }
        int[] a = new int[26];
        int[] b = new int[26];
        for (int i = 0; i < m; i++) {
            a[s1.charAt(i) - 'a']++;
            b[s2.charAt(i) - 'a']++;
        }
        if (Arrays.equals(a, b)) {
            return true;
        }
        int left = 0;
        for (int right = m; right < n; right++) {
            b[s2.charAt(right) - 'a']++;
            b[s2.charAt(left) - 'a']--;
            if (Arrays.equals(a, b)) {
                return true;
            }
            left++;
        }
        return false;
    }
}
```



# 验证回文串Ⅱ

## 题目描述

给定一个非空字符串 `s`，请判断如果 **最多** 从字符串中删除一个字符能否得到一个回文字符串。

**示例 1：**

```
输入: s = "aba"
输出: true
```

**示例 2：**

```
输入: s = "abca"
输出: true
解释: 可以删除 "c" 字符 或者 "b" 字符
```

**示例 3：**

```
输入: s = "abc"
输出: false
```

 

## 题解

当判断每个前后字符相等的时候就继续，如果不等于则判断删除前面的字符或者后面的字符，看剩下的是否能构成回文字符串，如果其中一个是回文字符串则可以通过删除一个字符实现

```java
class Solution {
    public boolean validPalindrome(String s) {
        int n = s.length();
        boolean flag = true;
        for(int left = 0, right = n - 1; left < right; left++, right--) {
            if(s.charAt(left) != s.charAt(right)) {
                return valid(s, left + 1, right) || valid(s, left, right - 1);
            }
        }
        return true;
    }

    public boolean valid(String s, int i, int j) {
        for(int left = i, right = j; left < right; left++, right--) {
            if(s.charAt(left) != s.charAt(right)) {
                return false;
            }
        }
        return true;
    }
}
```

