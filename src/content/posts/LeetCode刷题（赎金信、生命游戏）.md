---
title: LeetCode刷题（赎金信、生命游戏）
published: 2025-11-01
updated: 2025-11-01
description: '赎金信、生命游戏'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 赎金信

## 题目描述

给你两个字符串：`ransomNote` 和 `magazine` ，判断 `ransomNote` 能不能由 `magazine` 里面的字符构成。

如果可以，返回 `true` ；否则返回 `false` 。

`magazine` 中的每个字符只能在 `ransomNote` 中使用一次。

**示例 1：**

```
输入：ransomNote = "a", magazine = "b"
输出：false
```

**示例 2：**

```
输入：ransomNote = "aa", magazine = "ab"
输出：false
```

**示例 3：**

```
输入：ransomNote = "aa", magazine = "aab"
输出：true
```



## 题解

是之前最小覆盖字串的mini版本，也是用同样的思路用count计数

```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        Map<Character, Integer> map = new HashMap<>();
        int count = 0;
        for(char c : ransomNote.toCharArray()) {
            if(map.containsKey(c)) {
                map.put(c, map.get(c) + 1);
            } else {
                map.put(c, 1);
                count++;
            }
        }
        for(char c : magazine.toCharArray()) {
            if(map.containsKey(c)) {
                if(map.get(c) == 1) {
                    count--;
                    map.remove(c);
                } else {
                    map.put(c, map.get(c) - 1);
                }
            }
        }
        return count == 0;
    }
}
```



# 生命游戏

## 题目描述

根据 [百度百科](https://baike.baidu.com/item/生命游戏/2926434?fr=aladdin) ， **生命游戏** ，简称为 **生命** ，是英国数学家约翰·何顿·康威在 1970 年发明的细胞自动机。

给定一个包含 `m × n` 个格子的面板，每一个格子都可以看成是一个细胞。每个细胞都具有一个初始状态： `1` 即为 **活细胞** （live），或 `0` 即为 **死细胞** （dead）。每个细胞与其八个相邻位置（水平，垂直，对角线）的细胞都遵循以下四条生存定律：

1. 如果活细胞周围八个位置的活细胞数少于两个，则该位置活细胞死亡；
2. 如果活细胞周围八个位置有两个或三个活细胞，则该位置活细胞仍然存活；
3. 如果活细胞周围八个位置有超过三个活细胞，则该位置活细胞死亡；
4. 如果死细胞周围正好有三个活细胞，则该位置死细胞复活；

下一个状态是通过将上述规则同时应用于当前状态下的每个细胞所形成的，其中细胞的出生和死亡是 **同时** 发生的。给你 `m x n` 网格面板 `board` 的当前状态，返回下一个状态。

给定当前 `board` 的状态，**更新** `board` 到下一个状态。

**注意** 你不需要返回任何东西。

 

**示例 1：**

![339](../images/339.jpg)

```
输入：board = [[0,1,0],[0,0,1],[1,1,1],[0,0,0]]
输出：[[0,0,0],[1,0,1],[0,1,1],[0,1,0]]
```

**示例 2：**

![340](../images/340.jpg)

```
输入：board = [[1,1],[1,0]]
输出：[[1,1],[1,1]]
```

 

## 题解

八个方向计数，依次判断

```java
class Solution {
    public void gameOfLife(int[][] board) {
        int row = board.length;
        int col = board[0].length;
        int[][] result = new int[row][col];
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (judgeCode(board, i, j, row, col)) {
                    result[i][j] = 1;
                } else {
                    result[i][j] = 0;
                }
            }
        }
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                board[i][j] = result[i][j];
            }
        }
    }

    public boolean judgeCode(int[][] board, int i, int j, int row, int col) {
        int count = 0;
        if (i - 1 >= 0 && j - 1 >= 0)
            count += board[i - 1][j - 1];
        if (i - 1 >= 0)
            count += board[i - 1][j];
        if (i - 1 >= 0 && j + 1 < col)
            count += board[i - 1][j + 1];
        if (j + 1 < col)
            count += board[i][j + 1];
        if (i + 1 < row && j + 1 < col)
            count += board[i + 1][j + 1];
        if (i + 1 < row)
            count += board[i + 1][j];
        if (i + 1 < row && j - 1 >= 0)
            count += board[i + 1][j - 1];
        if (j - 1 >= 0)
            count += board[i][j - 1];
        if (board[i][j] == 1) {
            if (count == 2 || count == 3) {
                return true;
            } else {
                return false;
            }
        } else {
            if (count == 3) {
                return true;
            }
        }
        return false;
    }
}
```

