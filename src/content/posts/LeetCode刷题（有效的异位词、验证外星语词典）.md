---
title: LeetCode刷题（有效的字母异位词、验证外星语词典）
published: 2025-12-08
updated: 2025-12-08
description: '有效的字母异位词、验证外星语词典'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 有效的字母异位词

## 题目描述

给定两个字符串 `s` 和 `t` ，编写一个函数来判断它们是不是一组变位词（字母异位词）。

**注意：**若 `*s*` 和 `*t*` 中每个字符出现的次数都相同且**字符顺序不完全相同**，则称 `*s*` 和 `*t*` 互为变位词（字母异位词）。

**示例 1：**

```
输入：s = "anagram", t = "nagaram"
输出：true
```

**示例 2：**

```
输入：s = "rat", t = "car"
输出：false
```

**示例 3：**

```
输入：s = "a", t = "a"
输出：false
```



## 题解

采用的哈希表进行求解的，其实也可以用26位Int数组，不过为了可以存所有的字符还是用HashMap会好一点，遍历两个字符串，对每一个字符维护value（对应其数量），当数量为0的时候删除，最后判断是否完全匹配（map是否为0）

```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if(s.equals(t)) return false;
        Map<Character, Integer> map = new HashMap<>();
        for(char c : s.toCharArray()) {
            if(map.containsKey(c)) {
                map.put(c, map.get(c) + 1);
            } else {
                map.put(c, 1);
            }
        }

        for(char c : t.toCharArray()) {
            if(!map.containsKey(c)) {
                return false;
            } else {
                if(map.get(c) == 1) {
                    map.remove(c);
                } else {
                    map.put(c, map.get(c) - 1);
                }
            }
        }
        return map.size() == 0;
    }
}
```



# 验证外星语词典

## 题目描述

某种外星语也使用英文小写字母，但可能顺序 `order` 不同。字母表的顺序（`order`）是一些小写字母的排列。

给定一组用外星语书写的单词 `words`，以及其字母表的顺序 `order`，只有当给定的单词在这种外星语中按字典序排列时，返回 `true`；否则，返回 `false`。

**示例 1：**

```
输入：words = ["hello","leetcode"], order = "hlabcdefgijkmnopqrstuvwxyz"
输出：true
解释：在该语言的字母表中，'h' 位于 'l' 之前，所以单词序列是按字典序排列的。
```

**示例 2：**

```
输入：words = ["word","world","row"], order = "worldabcefghijkmnpqstuvxyz"
输出：false
解释：在该语言的字母表中，'d' 位于 'l' 之后，那么 words[0] > words[1]，因此单词序列不是按字典序排列的。
```

**示例 3：**

```
输入：words = ["apple","app"], order = "abcdefghijklmnopqrstuvwxyz"
输出：false
解释：当前三个字符 "app" 匹配时，第二个字符串相对短一些，然后根据词典编纂规则 "apple" > "app"，因为 'l' > '∅'，其中 '∅' 是空白字符，定义为比任何其他字符都小（更多信息）。
```



## 题解

题目描述也是外星语说是，后面说的order是字典序对应关系，把每个位置的字符index存储map中，在比较的时候直接从map中取出位置进行比较即可，注意如果前缀匹配的时候，后续长度大的要放在后边

```java
class Solution {
    public boolean isAlienSorted(String[] words, String order) {
        Map<Character, Integer> map = new HashMap<>();
        int i = 1;
        for(char c : order.toCharArray()) {
            map.put(c, i++);
        }
        int n = words.length;
        for(int k = 0; k < n - 1; k++) {
            String str1 = words[k];
            String str2 = words[k + 1];
            i = 0;
            while(i < str1.length() && i < str2.length()) {
                char c1 = str1.charAt(i);
                char c2 = str2.charAt(i);
                if(map.get(c1) < map.get(c2)) {
                    break;
                } else if (map.get(c1) > map.get(c2)) {
                    return false;
                } else {
                    i++;
                }
            }
            if((i == str1.length() || i == str2.length()) && str1.length() > str2.length()) {
                return false;
            }
        }  
        return true;
    }
}
```

