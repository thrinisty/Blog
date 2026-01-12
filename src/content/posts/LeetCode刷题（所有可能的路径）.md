---
title: LeetCode刷题（所有可能的路径）
published: 2026-01-11
updated: 2026-01-11
description: '所有可能的路径'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 所有可能的路径

## 题目描述

给定一个有 `n` 个节点的有向无环图，用二维数组 `graph` 表示，请找到所有从 `0` 到 `n-1` 的路径并输出（不要求按顺序）。

`graph` 的第 `i` 个数组中的单元都表示有向图中 `i` 号节点所能到达的下一些结点（译者注：有向图是有方向的，即规定了 a→b 你就不能从 b→a ），若为空，就是没有下一个节点了。

**示例 1：**

![519](../images/519.jpg)

```
输入：graph = [[1,2],[3],[3],[]]
输出：[[0,1,3],[0,2,3]]
解释：有两条路径 0 -> 1 -> 3 和 0 -> 2 -> 3
```

**示例 2：**

![520](../images/520.jpg)

```
输入：graph = [[4,3,1],[3,2,4],[3],[4],[]]
输出：[[0,4],[0,3,4],[0,1,3,4],[0,1,2,3,4],[0,1,4]]
```

**示例 3：**

```
输入：graph = [[1],[]]
输出：[[0,1]]
```

**示例 4：**

```
输入：graph = [[1,2,3],[2],[3],[]]
输出：[[0,1,2,3],[0,2,3],[0,3]]
```

**示例 5：**

```
输入：graph = [[1,3],[2],[3],[]]
输出：[[0,1,2,3],[0,3]]
```



## 题解

运用回溯法，依次尝试将下一个节点放入list中尝试下一步，直到最后一个节点被遍历到放入result中

```java
class Solution {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> list = new ArrayList<>();
    public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
        dfs(0, graph);
        return result;
    }

    public void dfs(int node, int[][] graph) {
        list.add(node);
        if(node == graph.length - 1) {
            result.add(new ArrayList<>(list));
        } else {
            for(int next : graph[node]) {
                dfs(next, graph);
            }
        }
        list.remove(list.size() - 1);
    }
}
```

