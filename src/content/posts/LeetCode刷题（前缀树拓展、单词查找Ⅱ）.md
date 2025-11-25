---
title: LeetCode刷题（前缀树拓展、单词查找Ⅱ）
published: 2025-11-16
updated: 2025-11-16
description: '添加与搜索单词、单词查找Ⅱ（超时）'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 添加与搜索单词

## 题目描述

请你设计一个数据结构，支持 添加新单词 和 查找字符串是否与任何先前添加的字符串匹配 。

实现词典类 `WordDictionary` ：

- `WordDictionary()` 初始化词典对象
- `void addWord(word)` 将 `word` 添加到数据结构中，之后可以对它进行匹配
- `bool search(word)` 如果数据结构中存在字符串与 `word` 匹配，则返回 `true` ；否则，返回 `false` 。`word` 中可能包含一些 `'.'` ，每个 `.` 都可以表示任何一个字母。

**示例：**

```
输入：
["WordDictionary","addWord","addWord","addWord","search","search","search","search"]
[[],["bad"],["dad"],["mad"],["pad"],["bad"],[".ad"],["b.."]]
输出：
[null,null,null,null,false,true,true,true]

解释：
WordDictionary wordDictionary = new WordDictionary();
wordDictionary.addWord("bad");
wordDictionary.addWord("dad");
wordDictionary.addWord("mad");
wordDictionary.search("pad"); // 返回 False
wordDictionary.search("bad"); // 返回 True
wordDictionary.search(".ad"); // 返回 True
wordDictionary.search("b.."); // 返回 True
```



## 题解

这一道题的添加思路和Node结构与前缀树的一致，难点是如何匹配 "." 的时候遍历其下的所有情况，我们采用dfs的方式进行遍历，如果c不为 "." 则按正常的方式匹配下一个字符，直到匹配完全，如果位 "." 的情况则需要遍历26个分叉，看是否有满足后续的可能出现，否则返回false

```java
class WordDictionary {

    class Node {
        boolean isEnd = false;
        Node[] next = new Node[26];
    }

    Node root;

    public WordDictionary() {
        root = new Node();
    }
    
    public void addWord(String word) {
        Node p = root;
        char[] arr = word.toCharArray();
        for(char c : arr) {
            int index = c - 'a';
            if(p.next[index] == null) {
                p.next[index] = new Node();
            }
            p = p.next[index];
        }
        p.isEnd = true;
    }
    
    public boolean search(String word) {
        return dfs(word, 0, root);
    }

    public boolean dfs(String word, int index, Node node) {
        if(index == word.length()) {
            return node.isEnd;
        }
        char c = word.charAt(index);
        if(c != '.') {
            int i = c - 'a';
            if(node.next[i] == null) return false;
            return dfs(word, index + 1, node.next[i]);
        } else {
            for(int i = 0; i < 26; i++) {
                if(node.next[i] != null && dfs(word, index + 1, node.next[i])) {
                    return true;
                }
            }
            return false;
        }
    }
}
```



# 单词查找Ⅱ

## 题目描述

给定一个 `m x n` 二维字符网格 `board` 和一个单词（字符串）列表 `words`， *返回所有二维网格上的单词* 。

单词必须按照字母顺序，通过 **相邻的单元格** 内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母在一个单词中不允许被重复使用。

**示例 1：**

![377](../images/377.jpg)

```
输入：board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]], words = ["oath","pea","eat","rain"]
输出：["eat","oath"]
```

**示例 2：**

![379](../images/379.jpg)

```
输入：board = [["a","b"],["c","d"]], words = ["abcb"]
输出：[]
```



## 题解

暴力解法：和之前的单词查找一致，看是否有满足的情况，如果有将word加入到结果中，最后返回结果

超时（63/65），之后有时间二刷再看看

```java
class Solution {
    public List<String> findWords(char[][] board, String[] words) {
        Set<String> result = new HashSet<>();
        boolean[][] visited = new boolean[board.length][board[0].length];
        for(int i = 0; i < board.length; i++) {
            for(int j = 0; j < board[0].length; j++) {
                for(String word : words) {
                    if(dfs(board, visited, word, i, j, 0)) {
                        result.add(word);
                    }
                }
            }
        }
        return new ArrayList<>(result);
    }

    public boolean dfs(char[][] board, boolean[][] visited, String word, int i, int j, int index) {
        if(i >= 0 && i < board.length && j >= 0 && j < board[0].length && !visited[i][j]) {
            if(word.charAt(index) != board[i][j]) {
                return false;
            }
            if(index == word.length() - 1) {
                return true;
            }
            visited[i][j] = true;
            boolean u = dfs(board, visited, word, i - 1, j, index + 1);
            boolean d = dfs(board, visited, word, i + 1, j, index + 1);
            boolean l = dfs(board, visited, word, i, j - 1, index + 1);
            boolean r = dfs(board, visited, word, i, j + 1, index + 1);
            visited[i][j] = false;
            return u || d || l || r;
        } else {
            return false;
        }
    }
}
```

