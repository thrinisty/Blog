---
title: LeetCode刷题（找出匹配项、验证回文串）
published: 2025-10-28
updated: 2025-10-28
description: '找出字符串中第一个匹配项下表、验证回文串'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 找出匹配项

## 题目描述

给你两个字符串 `haystack` 和 `needle` ，请你在 `haystack` 字符串中找出 `needle` 字符串的第一个匹配项的下标（下标从 0 开始）。如果 `needle` 不是 `haystack` 的一部分，则返回 `-1` 。

**示例 1：**

```
输入：haystack = "sadbutsad", needle = "sad"
输出：0
解释："sad" 在下标 0 和 6 处匹配。
第一个匹配项的下标是 0 ，所以返回 0 。
```

**示例 2：**

```
输入：haystack = "leetcode", needle = "leeto"
输出：-1
解释："leeto" 没有在 "leetcode" 中出现，所以返回 -1 。
```



## 题解

取出对应长度的区间依次比较即可

```java
class Solution {
    public int strStr(String haystack, String needle) {
        int length = needle.length();
        for(int i = 0; i < haystack.length() - length + 1; i++) {
            String target = haystack.substring(i, i + length);
            if(target.equals(needle)) {
                return i;
            }
        }
        return -1;
    }
}
```



# 验证回文串

## 题目描述

如果在将所有大写字符转换为小写字符、并移除所有非字母数字字符之后，短语正着读和反着读都一样。则可以认为该短语是一个 **回文串** 。

字母和数字都属于字母数字字符。

给你一个字符串 `s`，如果它是 **回文串** ，返回 `true` ；否则，返回 `false` 。

**示例 1：**

```
输入: s = "A man, a plan, a canal: Panama"
输出：true
解释："amanaplanacanalpanama" 是回文串。
```

**示例 2：**

```
输入：s = "race a car"
输出：false
解释："raceacar" 不是回文串。
```

**示例 3：**

```
输入：s = " "
输出：true
解释：在移除非字母数字字符之后，s 是一个空字符串 "" 。
由于空字符串正着反着读都一样，所以是回文串。
```



## 题解

```java
class Solution {
    public boolean isPalindrome(String s) {
        List<Character> list = new ArrayList<>();
        for(char c : s.toCharArray()) {
            if(c >= 'A' && c <= 'Z') {
                list.add((char)(c + 32));
            }else if(c >= 'a' && c <= 'z') {
                list.add(c);
            }else if(c >= '0' && c <= '9') {
                list.add(c);
            }
        }
        int i = 0;
        int j = list.size() - 1;
        while(i < j) {
            if(list.get(i) != list.get(j)) {
                return false;
            }
            i++;
            j--;
        }
        return true;
    }
}
```

