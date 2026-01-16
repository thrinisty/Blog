---
title: LeetCode刷题（相似字符串组）
published: 2026-01-14
updated: 2026-01-14
description: '相似字符串组'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 相似字符串组

## 题目描述

如果交换字符串 `X` 中的两个不同位置的字母，使得它和字符串 `Y` 相等，那么称 `X` 和 `Y` 两个字符串相似。如果这两个字符串本身是相等的，那它们也是相似的。

例如，`"tars"` 和 `"rats"` 是相似的 (交换 `0` 与 `2` 的位置)； `"rats"` 和 `"arts"` 也是相似的，但是 `"star"` 不与 `"tars"`，`"rats"`，或 `"arts"` 相似。

总之，它们通过相似性形成了两个关联组：`{"tars", "rats", "arts"}` 和 `{"star"}`。注意，`"tars"` 和 `"arts"` 是在同一组中，即使它们并不相似。形式上，对每个组而言，要确定一个单词在组中，只需要这个词和该组中至少一个单词相似。

给定一个字符串列表 `strs`。列表中的每个字符串都是 `strs` 中其它所有字符串的一个 **字母异位词** 。请问 `strs` 中有多少个相似字符串组？

**字母异位词（anagram）**，一种把某个字符串的字母的位置（顺序）加以改换所形成的新词。

**示例 1：**

```
输入：strs = ["tars","rats","arts","star"]
输出：2
```

**示例 2：**

```
输入：strs = ["omv","ovm"]
输出：1
```



## 题解

运用并交集进行求解，对于每个组合判断其是否是满足check交换位置后相等，如果相等则判断为是同一组，进行合并，之后求联通分量即为结果（组数）

```java
class Solution {
    int[] parent;
    public int numSimilarGroups(String[] strs) {
        int n = strs.length;
        parent = new int[n];
        for(int i = 0; i < n; i++) {
            parent[i] = i;
        }
        int result = 0;
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < i; j++) {
                if(check(strs[i], strs[j])) {
                    union(i, j);
                }
            }
        }
        for(int i = 0; i < n; i++) {
            if(parent[i] == i) {
                result++;
            }
        }
        return result;
    }

    public void union(int x, int y) {
        parent[find(x)] = find(y);
    }

    public int find(int x) {
        if(parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    public boolean check(String s1, String s2) {
        int count = 0;
        for(int i = 0; i < s1.length(); i++) {
            if(s1.charAt(i) != s2.charAt(i)) {
                count++;
                if(count > 2) {
                    return false;
                }
            }
        }
        return true;
    }
}
```

