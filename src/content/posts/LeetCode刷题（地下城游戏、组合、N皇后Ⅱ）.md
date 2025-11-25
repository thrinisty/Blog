---
title: LeetCode刷题（地下城游戏、组合、N皇后Ⅱ）
published: 2025-11-17
updated: 2025-11-17
description: '地下城游戏、组合、N皇后Ⅱ'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 地下城游戏

## 题目描述

恶魔们抓住了公主并将她关在了地下城 `dungeon` 的 **右下角** 。地下城是由 `m x n` 个房间组成的二维网格。我们英勇的骑士最初被安置在 **左上角** 的房间里，他必须穿过地下城并通过对抗恶魔来拯救公主。

骑士的初始健康点数为一个正整数。如果他的健康点数在某一时刻降至 0 或以下，他会立即死亡。

有些房间由恶魔守卫，因此骑士在进入这些房间时会失去健康点数（若房间里的值为*负整数*，则表示骑士将损失健康点数）；其他房间要么是空的（房间里的值为 *0*），要么包含增加骑士健康点数的魔法球（若房间里的值为*正整数*，则表示骑士将增加健康点数）。

为了尽快解救公主，骑士决定每次只 **向右** 或 **向下** 移动一步。

返回确保骑士能够拯救到公主所需的最低初始健康点数。

**注意：**任何房间都可能对骑士的健康点数造成威胁，也可能增加骑士的健康点数，包括骑士进入的左上角房间以及公主被监禁的右下角房间。

 

**示例 1：**

![380](../images/380.jpg)

```
输入：dungeon = [[-2,-3,3],[-5,-10,1],[10,30,-5]]
输出：7
解释：如果骑士遵循最佳路径：右 -> 右 -> 下 -> 下 ，则骑士的初始健康点数至少为 7 。
```



## 题解

从右下角开始运用动态规划求解，dp代表从当前格出发至少需要多少生命值才可以到达右下角

```java
class Solution {
    public int calculateMinimumHP(int[][] dungeon) {
        int row = dungeon.length;
        int col = dungeon[0].length;
        int[][] dp = new int[row][col];
        dp[row - 1][col - 1] = Math.max(1, 1 - dungeon[row - 1][col - 1]);
        for(int i = row - 2; i >= 0; i--) {
            dp[i][col - 1] = Math.max(1, dp[i + 1][col - 1] - dungeon[i][col - 1]);
        }
        for(int j = col - 2; j >= 0; j--) {
            dp[row - 1][j] = Math.max(1, dp[row - 1][j + 1] - dungeon[row - 1][j]);
        }
        for(int i = row - 2; i >= 0; i--) {
            for(int j = col - 2; j >= 0; j--) {
                int min = Math.min(dp[i + 1][j], dp[i][j + 1]);
                dp[i][j] = Math.max(1, min - dungeon[i][j]);
            }
        }
        return dp[0][0];
    }
}
```



# 组合

## 题目描述

给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。

你可以按 **任何顺序** 返回答案。

**示例 1：**

```
输入：n = 4, k = 2
输出：
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```

**示例 2：**

```
输入：n = 1, k = 1
输出：[[1]]
```



## 题解

运用回溯法，记录下标，对下标到最右侧的数字可以被选取，当长度位k的时候将结果加入到result中

```java
class Solution {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> list = new ArrayList<>();
    public List<List<Integer>> combine(int n, int k) {
        dfs(n, k, 1);
        return result;
    }

    public void dfs(int n, int k, int index) {
        if(list.size() == k) {
            result.add(new ArrayList<>(list));
            return;
        }
        for(int i = index; i <= n; i++) {
            list.add(i);
            dfs(n, k, i + 1);
            list.remove(list.size() - 1);
        }
    }
}
```



# N皇后

## 题目描述

**n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n × n` 的棋盘上，并且使皇后彼此之间不能相互攻击。

给你一个整数 `n` ，返回 **n 皇后问题** 不同的解决方案的数量。



## 题解

给定map用于存储每个行上放置的列位置，依次放置每一行，对于每一行的每一列，如果可以放置则递归的放置下一层

```java
class Solution {
    int result = 0;
    public int totalNQueens(int n) {
        int[] map = new int[n];
        func(map, 0);
        return result;
    }

    public void func(int[] map, int level) {
        int n = map.length;
        if(level == n) {
            result++;
            return;
        }
        for(int i = 0; i < n; i++) {
            map[level] = i;
            if(canPlace(map, level, i)) {
                func(map, level + 1);
            } 
        }
    }

    public boolean canPlace(int[] map, int row, int col) {
        for(int i = 0; i < row; i++) {
            if(col == map[i] || row + col == i + map[i] || row - col == i - map[i]) {
                return false;
            }
        }
        return true;
    }
}
```

