---
title: LeetCode刷题（岛屿的最大面积）
published: 2026-01-03
updated: 2026-01-03
description: '岛屿的最大面积'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 岛屿的最大面积

## 题目描述

给定一个由 `0` 和 `1` 组成的非空二维数组 `grid` ，用来表示海洋岛屿地图。

一个 **岛屿** 是由一些相邻的 `1` (代表土地) 构成的组合，这里的「相邻」要求两个 `1` 必须在水平或者竖直方向上相邻。你可以假设 `grid` 的四个边缘都被 `0`（代表水）包围着。

找到给定的二维数组中最大的岛屿面积。如果没有岛屿，则返回面积为 `0` 。

 

**示例 1：**

![512](../images/512.png)

```
输入: grid = 
[[0,0,1,0,0,0,0,1,0,0,0,0,0],[0,0,0,0,0,0,0,1,1,1,0,0,0],[0,1,1,0,1,0,0,0,0,0,0,0,0],[0,1,0,0,1,1,0,0,1,0,1,0,0],[0,1,0,0,1,1,0,0,1,1,1,0,0],[0,0,0,0,0,0,0,0,0,0,1,0,0],[0,0,0,0,0,0,0,1,1,1,0,0,0],[0,0,0,0,0,0,0,1,1,0,0,0,0]]
输出: 6
解释: 对于上面这个给定矩阵应返回 6。注意答案不应该是 11 ，因为岛屿只能包含水平或垂直的四个方向的 1 。
```

**示例 2：**

```
输入: grid = [[0,0,0,0,0,0,0,0]]
输出: 0
```



## 题解

运用递归进行求解，当满足条件的时候计算上下左右的贡献，将其相加，再加上本格的结果1返回即可，在主函数种迭代result岛屿最大面积即可

```java
class Solution {
    public int maxAreaOfIsland(int[][] grid) {
        int result = 0;
        for(int i = 0; i < grid.length; i++) {
            for(int j = 0; j < grid[0].length; j++) {
                if(grid[i][j] == 1) {
                    result = Math.max(result, dfs(grid, i, j));
                }
            }
        }
        return result;
    }

    public int dfs(int[][] grid, int i, int j) {
        if(i >= 0 && i < grid.length && j >= 0 && j < grid[0].length && grid[i][j] == 1) {
            grid[i][j] = 0;
            int up = dfs(grid, i - 1, j);
            int down = dfs(grid, i + 1, j);
            int left = dfs(grid, i, j - 1);
            int right = dfs(grid, i, j + 1);
            return 1 + up + down + left + right;
        }
        return 0;
    }
}
```

