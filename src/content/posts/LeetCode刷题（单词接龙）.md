---
title: LeetCode刷题（单词接龙）
published: 2026-01-08
updated: 2026-01-08
description: '单词接龙'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 单词接龙

## 题目描述

在字典（单词列表） `wordList` 中，从单词 `beginWord` 和 `endWord` 的 **转换序列** 是一个按下述规格形成的序列：

- 序列中第一个单词是 `beginWord` 。
- 序列中最后一个单词是 `endWord` 。
- 每次转换只能改变一个字母。
- 转换过程中的中间单词必须是字典 `wordList` 中的单词。

给定两个长度相同但内容不同的单词 `beginWord` 和 `endWord` 和一个字典 `wordList` ，找到从 `beginWord` 到 `endWord` 的 **最短转换序列** 中的 **单词数目** 。如果不存在这样的转换序列，返回 0。

**示例 1：**

```
输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
输出：5
解释：一个最短转换序列是 "hit" -> "hot" -> "dot" -> "dog" -> "cog", 返回它的长度 5。
```

**示例 2：**

```
输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log"]
输出：0
解释：endWord "cog" 不在字典中，所以无法进行转换。
```



## 题解

用edge存放可相互转化的边，并将包含开始字符串的字符存到words中，用广度搜索，直到找到目标的字符串为止（即target），每个从队列中出队的元素要找所有的与之有边的字符位置加入队列（并且这个为止没有被访问过）用step记录长度，当访问到目标的时候返回step + 1即可

```java
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        int n = wordList.size();
        String[] words = new String[n + 1];
        words[0] = beginWord;
        int target = -1;
        for(int i = 0; i < n; i++) {
            words[i + 1] = wordList.get(i);
            if(endWord.equals(wordList.get(i))) {
                target = i + 1;
            }
        }
        if(target == -1) {
            return 0;
        }
        boolean[][] edge = new boolean[n + 1][n + 1];
        boolean[] visited = new boolean[n + 1];
        for(int i = 1; i <= n; i++) {
            for(int j = 0; j <= i; j++) {
                if(canChange(words[i], words[j])) {
                    edge[i][j] = true;
                    edge[j][i] = true;
                }
            }
        }
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(0);
        visited[0] = true;
        int step = 1;
        while(!queue.isEmpty()) {
            int len = queue.size();
            for(int k = 0; k < len; k++) {
                int i = queue.poll();
                for(int j = 0; j <= n; j++) {
                    if(!visited[j] && edge[i][j]) {
                        if(j == target) {
                            return step + 1;
                        }
                        queue.offer(j);
                        visited[j] = true;
                    }
                }
            }
            step++;
        }
        return 0;
    }

    public boolean canChange(String s1, String s2) {
        int n = s1.length();
        int count = 0;
        for(int i = 0; i < n; i++) {
            if(s1.charAt(i) != s2.charAt(i)) {
                count++;
                if(count > 1) {
                    return false;
                }
            }
        }
        return count == 1;
    }
}
```

