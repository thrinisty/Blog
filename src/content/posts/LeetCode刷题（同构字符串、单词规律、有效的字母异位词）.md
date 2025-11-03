---
title: LeetCode刷题（同构字符串、单词规律、有效的字母异位词）
published: 2025-11-02
updated: 2025-11-02
description: '同构字符串、单词规律、有效的字母异位词'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 同构字符串

## 题目描述

给定两个字符串 `s` 和 `t` ，判断它们是否是同构的。

如果 `s` 中的字符可以按某种映射关系替换得到 `t` ，那么这两个字符串是同构的。

每个出现的字符都应当映射到另一个字符，同时不改变字符的顺序。不同字符不能映射到同一个字符上，相同字符只能映射到同一个字符上，字符可以映射到自己本身。

**示例 1:**

```
输入：s = "egg", t = "add"
输出：true
```

**示例 2：**

```
输入：s = "foo", t = "bar"
输出：false
```

**示例 3：**

```
输入：s = "paper", t = "title"
输出：true
```



## 题解

要保证不同的字符不能映射到同一字符上需要维护反向的映射关系

```java
class Solution {
    public boolean isIsomorphic(String s, String t) {
        HashMap<Character, Character> smapt = new HashMap<>();
        HashMap<Character, Character> tmaps = new HashMap<>();
        int n = s.length();
        int m = t.length();
        if(n != m) {
            return false;
        }
        for(int i = 0; i < n; i++) {
            char c1 = s.charAt(i);
            char c2 = t.charAt(i);
            if(smapt.containsKey(c1)) {
                char target = smapt.get(c1);
                if(target != c2) {
                    return false;
                }
            } else {
                smapt.put(c1, c2);
            }
            if(tmaps.containsKey(c2)) {
                char target = tmaps.get(c2);
                if(target != c1) {
                    return false;
                }
            } else {
                tmaps.put(c2, c1);
            }
        }
        return true;
    }
}
```



# 单词规律

## 题目描述

给定一种规律 `pattern` 和一个字符串 `s` ，判断 `s` 是否遵循相同的规律。

这里的 **遵循** 指完全匹配，例如， `pattern` 里的每个字母和字符串 `s` 中的每个非空单词之间存在着双向连接的对应规律。具体来说：

- `pattern` 中的每个字母都 **恰好** 映射到 `s` 中的一个唯一单词。
- `s` 中的每个唯一单词都 **恰好** 映射到 `pattern` 中的一个字母。
- 没有两个字母映射到同一个单词，也没有两个单词映射到同一个字母。

**示例1:**

```
输入: pattern = "abba", s = "dog cat cat dog"
输出: true
```

**示例 2:**

```
输入:pattern = "abba", s = "dog cat cat fish"
输出: false
```

**示例 3:**

```
输入: pattern = "aaaa", s = "dog cat cat dog"
输出: false
```



## 题解

和上一题思路一致，只不过映射结构换为了Character to String的形式

```java
class Solution {
    public boolean wordPattern(String pattern, String s) {
        Map<Character, String> ctos = new HashMap<>();
        Map<String, Character> stoc = new HashMap<>();
        String[] strs = s.split(" ");
        int m = pattern.length();
        int n = strs.length;
        if(m != n) return false;
        for(int i = 0; i < m; i++) {
            char c = pattern.charAt(i);
            String str = strs[i];
            if(ctos.containsKey(c)) {
                String target = ctos.get(c);
                if(!str.equals(target)) return false;
            } else {
                ctos.put(c, str);
            }
            if(stoc.containsKey(str)) {
                char target = stoc.get(str);
                if(c != target) return false;
            } else {
                stoc.put(str, c);
            }
        }
        return true;
    }
}
```



# 有效的字母异位词

## 题目描述

给定两个字符串 `s` 和 `t` ，编写一个函数来判断 `t` 是否是 `s` 的 字母异位词。

**示例 1:**

```
输入: s = "anagram", t = "nagaram"
输出: true
```

**示例 2:**

```
输入: s = "rat", t = "car"
输出: false
```



## 题解

通过count计数，将出现的字符以及其数量放入map集合，匹配另一个字符串注意及时没有匹配对应字符时返回false，最后看字符是否全部被匹配完成

```java
class Solution {
    public boolean isAnagram(String s, String t) {
        Map<Character, Integer> map = new HashMap<>();
        int n = s.length();
        int m = t.length();
        if(n != m) return false;
        int count = 0;
        for(char c : s.toCharArray()) {
            if(map.containsKey(c)) {
                map.put(c, map.get(c) + 1);
            } else {
                count++;
                map.put(c, 1);
            }
        }
        for(char c : t.toCharArray()) {
            if(map.containsKey(c)) {
                int num = map.get(c);
                if(num == 1) {
                    count--;
                    map.remove(c);
                } else {
                    map.put(c, map.get(c) - 1);
                }
            } else {
                return false;
            }
        }
        return count == 0;
    }
}
```

