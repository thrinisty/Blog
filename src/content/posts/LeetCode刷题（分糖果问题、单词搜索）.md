---
title: LeetCode刷题（分糖果问题、单词搜索）
published: 2025-10-14
updated: 2025-10-14
description: '分糖果问题、单词搜索'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 分糖果问题

## 题目描述

一群孩子做游戏，现在请你根据游戏得分来发糖果，要求如下：

\1. 每个孩子不管得分多少，起码分到一个糖果。

\2. 任意两个相邻的孩子之间，得分较多的孩子必须拿多一些糖果。(若相同则无此限制)

给定一个数组 arr*a**r**r* 代表得分数组，请返回最少需要多少糖果。

要求: 时间复杂度为 O(n)*O*(*n*) 空间复杂度为 O(n)*O*(*n*)

数据范围： 1≤n≤1000001≤*n*≤100000 ，1≤ai≤10001≤*a**i*≤1000



## 题解

贪心算法，从左向右如果右边的数大于左边，他需要的糖果就要在左边的基础上+1，从右往左，思路一致，但是要保留左边的糖果限制，不能比左得出的结果小，取最大的那个，最后将糖果加在一起

```java
public class Solution {
    public int candy (int[] arr) {
        // write code here
        int n = arr.length;
        int count[] = new int[n];
        Arrays.fill(count, 1);
        for(int i = 1; i < n; i++) {
            if(arr[i] > arr[i - 1]) {
                count[i] = count[i - 1] + 1;
            }
        }
        for(int i = n - 2; i >= 0; i--) {
            if(arr[i] > arr[i + 1]) {
                count[i] = Math.max(count[i], count[i + 1] + 1);
            }
        }
        int result = 0;
        for(int con : count) {
            result += con;
        }
        return result;
    }
}
```



# 单词搜索

## 题目描述

给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

**示例 1：**

![283](../images/283.jpg)

```
输入：board = [['A','B','C','E'],['S','F','C','S'],['A','D','E','E']], word = "ABCCED"
输出：true
```

**示例 2：**

![284](../images/284.jpg)

```
输入：board = [['A','B','C','E'],['S','F','C','S'],['A','D','E','E']], word = "SEE"
输出：true
```



## 题解

回溯法，通过遍历每个起始点，递归使用check看上下左右是否匹配后续的对应字符，不匹配返回false，再判断是否匹配到了最后一个字符，如果是返回true，没有则继续匹配上下左右的字符，只要有一种匹配结果匹配成功即可返回true

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        int row = board.length;
        int col = board[0].length;
        boolean[][] visited = new boolean[row][col];
        for(int i = 0; i < row; i++) {
            for(int j = 0; j < col; j++) {
                boolean flag = check(board, visited, i, j, word, 0);
                if(flag) {
                    return true;
                }
            }
        }
        return false;
    }

    public boolean check(char[][] board, boolean[][] visited, int i, int j, String word, int index) {
        if(i >= 0 && i < board.length && j >= 0 && j < board[0].length && !visited[i][j]) {
            if(word.charAt(index) != board[i][j]) {
                return false;
            }
            if(index == word.length() - 1) {
                return true;
            }
            visited[i][j] = true;
            boolean u = check(board, visited, i - 1, j, word, index + 1);
            boolean d = check(board, visited, i + 1, j, word, index + 1);
            boolean l = check(board, visited, i, j - 1, word, index + 1);
            boolean r = check(board, visited, i, j + 1, word, index + 1);
            visited[i][j] = false;
            return u || d || l || r;
        }
        return false;
    }
}
```

