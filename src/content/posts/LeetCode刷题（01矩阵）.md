---
title: LeetCode刷题（01矩阵）
published: 2026-01-07
updated: 2026-01-07
description: '01矩阵'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 01矩阵

## 题目描述

给定一个由 `0` 和 `1` 组成的矩阵 `mat` ，请输出一个大小相同的矩阵，其中每一个格子是 `mat` 中对应位置元素到最近的 `0` 的距离。

两个相邻元素间的距离为 `1`。

**示例 1：**

![517](../images/517.png)

```
输入：mat = [[0,0,0],[0,1,0],[0,0,0]]
输出：[[0,0,0],[0,1,0],[0,0,0]]
```

**示例 2：**

![518](../images/518.png)

```
输入：mat = [[0,0,0],[0,1,0],[1,1,1]]
输出：[[0,0,0],[0,1,0],[1,2,1]]
```



## 题解

正难则反，从每个0开始出发，进过的非0位置记录最近一个0到这里的距离，在运用队列初始将每个0放入，依次广搜即可

```java
class Solution {
    private int[][] arr = new int[][] { { -1, 0 }, { 1, 0 }, { 0, -1 }, { 0, 1 } };

    public int[][] updateMatrix(int[][] mat) {
        int m = mat.length;
        int n = mat[0].length;
        int[][] result = new int[m][n];
        boolean[][] visited = new boolean[m][n];
        Queue<int[]> queue = new LinkedList<>();
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (mat[i][j] == 0) {
                    queue.offer(new int[] { i, j });
                    visited[i][j] = true;
                }
            }
        }
        while (!queue.isEmpty()) {
            int[] place = queue.poll();
            int px = place[0];
            int py = place[1];
            for (int d = 0; d < arr.length; d++) {
                int i = px + arr[d][0];
                int j = py + arr[d][1];
                if (i >= 0 && i < m && j >= 0 && j < n && !visited[i][j]) {
                    result[i][j] = result[px][py] + 1;
                    queue.offer(new int[] { i, j });
                    visited[i][j] = true;
                }
            }
        }
        return result;
    }
}
```

