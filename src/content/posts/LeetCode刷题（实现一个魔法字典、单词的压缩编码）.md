---
title: LeetCode刷题（实现一个魔法字典、单词的压缩编码）
published: 2025-12-18
updated: 2025-12-18
description: '实现一个魔法字典、单词的压缩编码'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 实现一个魔法字典

## 题目描述

设计一个使用单词列表进行初始化的数据结构，单词列表中的单词 **互不相同** 。 如果给出一个单词，请判定能否只将这个单词中**一个**字母换成另一个字母，使得所形成的新单词存在于已构建的神奇字典中。

实现 `MagicDictionary` 类：

- `MagicDictionary()` 初始化对象
- `void buildDict(String[] dictionary)` 使用字符串数组 `dictionary` 设定该数据结构，`dictionary` 中的字符串互不相同
- `bool search(String searchWord)` 给定一个字符串 `searchWord` ，判定能否只将字符串中 **一个** 字母换成另一个字母，使得所形成的新字符串能够与字典中的任一字符串匹配。如果可以，返回 `true` ；否则，返回 `false` 。

**示例：**

```
输入
inputs = ["MagicDictionary", "buildDict", "search", "search", "search", "search"]
inputs = [[], [["hello", "leetcode"]], ["hello"], ["hhllo"], ["hell"], ["leetcoded"]]
输出
[null, null, false, true, false, false]

解释
MagicDictionary magicDictionary = new MagicDictionary();
magicDictionary.buildDict(["hello", "leetcode"]);
magicDictionary.search("hello"); // 返回 False
magicDictionary.search("hhllo"); // 将第二个 'h' 替换为 'e' 可以匹配 "hello" ，所以返回 True
magicDictionary.search("hell"); // 返回 False
magicDictionary.search("leetcoded"); // 返回 False
```



## 题解

运用前缀树求解，加入单词的部分与前缀树一致，差别在于查找需要对其中的一个字符进行替换，替换后从该字符下开始遍历后续的字符即contain函数，其中一种情况满足true即可，如果替换都不满足，则匹配当前字符向下继续判断（不匹配则返回flase）

```java
class MagicDictionary {
    class Node {
        boolean isEnd = false;
        Node[] next = new Node[26];
    }

    Node root = new Node();    
    public void buildDict(String[] dictionary) {
        for(String word : dictionary) {
            Node p = root;
            for(char c : word.toCharArray()) {
                int index = c - 'a';
                if(p.next[index] == null) {
                    p.next[index] = new Node();
                }
                p = p.next[index];
            }
            p.isEnd = true;
        }
    }
    
    public boolean search(String searchWord) {
        int n = searchWord.length();
        Node p = root;
        for(int i = 0; i < n; i++) {
            int index = searchWord.charAt(i) - 'a';
            for(int j = 0; j < 26; j++) {
                if(j == index || p.next[j] == null) {
                    continue;
                }
                if(contain(searchWord.substring(i + 1), p.next[j])) {
                    return true;
                }
            }
            if(p.next[index] == null) {
                return false;
            }
            p = p.next[index];
        }
        return false;
    }

    public boolean contain(String word, Node p) {
        for(char c : word.toCharArray()) {
            int index = c - 'a';
            if(p.next[index] == null) {
                return false;
            }
            p = p.next[index];
        }
        return p.isEnd;
    }
}
```



# 单词的压缩编码

## 题目描述

单词数组 `words` 的 **有效编码** 由任意助记字符串 `s` 和下标数组 `indices` 组成，且满足：

- `words.length == indices.length`
- 助记字符串 `s` 以 `'#'` 字符结尾
- 对于每个下标 `indices[i]` ，`s` 的一个从 `indices[i]` 开始、到下一个 `'#'` 字符结束（但不包括 `'#'`）的 **子字符串** 恰好与 `words[i]` 相等

给定一个单词数组 `words` ，返回成功对 `words` 进行编码的最小助记字符串 `s` 的长度 。

**示例 1：**

```
输入：words = ["time", "me", "bell"]
输出：10
解释：一组有效编码为 s = "time#bell#" 和 indices = [0, 2, 5] 。
words[0] = "time" ，s 开始于 indices[0] = 0 到下一个 '#' 结束的子字符串，如加粗部分所示 "time#bell#"
words[1] = "me" ，s 开始于 indices[1] = 2 到下一个 '#' 结束的子字符串，如加粗部分所示 "time#bell#"
words[2] = "bell" ，s 开始于 indices[2] = 5 到下一个 '#' 结束的子字符串，如加粗部分所示 "time#bell#"
```

**示例 2：**

```
输入：words = ["t"]
输出：2
解释：一组有效编码为 s = "t#" 和 indices = [0] 。
```



## 题解

将words从大到小按照长度排序，这样小的就可以被后边的大的word包含，遍历，依次判断是否存在满足set中有str以word结尾，如果有则不用放入，如果没有则需要放入set，并更新增加result（单词长度+#）

```java
class Solution {
    public int minimumLengthEncoding(String[] words) {
        Arrays.sort(words, (a, b) -> b.length() - a.length());
        Set<String> set = new HashSet<>();
        int result = 0;
        for(String word : words) {
            boolean flag = false;
            for(String str : set) {
                if(str.endsWith(word)) {
                    flag = true;
                    break;
                }
            }
            if(!flag) {
                set.add(word);
                result += word.length() + 1;
            }
        }
        return result;
    }
}
```

