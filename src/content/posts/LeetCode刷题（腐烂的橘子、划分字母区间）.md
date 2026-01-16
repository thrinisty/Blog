---
title: LeetCode刷题（腐烂的橘子、划分字母区间）
published: 2026-01-16
updated: 2026-01-16
description: '剑指117变种完结撒花，启程hot100，其实之前做过hot150剩下的也就只剩两题了，不过也再复习一遍'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 腐烂的橘子

## 题目描述

在给定的 `m x n` 网格 `grid` 中，每个单元格可以有以下三个值之一：

- 值 `0` 代表空单元格；
- 值 `1` 代表新鲜橘子；
- 值 `2` 代表腐烂的橘子。

每分钟，腐烂的橘子 **周围 4 个方向上相邻** 的新鲜橘子都会腐烂。

返回 *直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`* 。

 

**示例 1：**

![525](../images/525.png)

```
输入：grid = [[2,1,1],[1,1,0],[0,1,1]]
输出：4
```

**示例 2：**

```
输入：grid = [[2,1,1],[0,1,1],[1,0,1]]
输出：-1
解释：左下角的橘子（第 2 行， 第 0 列）永远不会腐烂，因为腐烂只会发生在 4 个方向上。
```

**示例 3：**

```
输入：grid = [[0,2]]
输出：0
解释：因为 0 分钟时已经没有新鲜橘子了，所以答案就是 0 。
```



## 题解

广度搜索，并维护一个time时间每次取出后time++，最后判断是否还有剩余的新鲜橘子，如果没有则返回time

```java
class Solution {
    int[][] arr = new int[][]{{1, 0}, {-1, 0}, {0, 1}, {0, -1}}; 
    public int orangesRotting(int[][] grid) {
        int row = grid.length;
        int col = grid[0].length;
        Queue<int[]> queue = new LinkedList<>();
        int rest = 0;
        for(int i = 0; i < row; i++) {
            for(int j = 0; j < col; j++) {
                if(grid[i][j] == 2) {
                    queue.offer(new int[]{i, j});
                } else if(grid[i][j] == 1) {
                    rest++;
                }
            }
        }
        int time = 0;
        while(!queue.isEmpty()) {
            if(rest == 0) {
                return time;
            }
            int n = queue.size();
            for(int i = 0; i < n; i++) {
                int[] cur = queue.poll();
                for(int j = 0; j < 4; j++) {
                    int x = cur[0] + arr[j][0];
                    int y = cur[1] + arr[j][1];
                    if(x >= 0 && x < row && y >= 0 && y < col && grid[x][y] == 1) {
                        rest--;
                        grid[x][y] = 2;
                        queue.offer(new int[]{x, y});
                    }
                } 
            }
            time++;
        }
        return rest == 0 ? time : -1;
    }
}
```



# 划分字母区间

## 题目描述

给你一个字符串 `s` 。我们要把这个字符串划分为尽可能多的片段，同一字母最多出现在一个片段中。例如，字符串 `"ababcc"` 能够被分为 `["abab", "cc"]`，但类似 `["aba", "bcc"]` 或 `["ab", "ab", "cc"]` 的划分是非法的。

注意，划分结果需要满足：将所有划分结果按顺序连接，得到的字符串仍然是 `s` 。

返回一个表示每个字符串片段的长度的列表。

**示例 1：**

```
输入：s = "ababcbacadefegdehijhklij"
输出：[9,7,8]
解释：
划分结果为 "ababcbaca"、"defegde"、"hijhklij" 。
每个字母最多出现在一个片段中。
像 "ababcbacadefegde", "hijhklij" 这样的划分是错误的，因为划分的片段数较少。 
```

**示例 2：**

```
输入：s = "eccbbbbdec"
输出：[10]
```



## 题解

和跳跃游戏有些类似，都是使用贪心算法进行求解：对于每个字符用last数组存储其最后一个位置，在第二次遍历的循环中维护左右标识，遍历到每个字符迭代结束的位置，当遍历到右标识的时候，此时维护的区间的各个字符都不存在于后续位置，即可拆分

```java
class Solution {
    public List<Integer> partitionLabels(String s) {
        int[] last = new int[26];
        int n = s.length();
        for(int i = 0; i < n; i++) {
            last[s.charAt(i) - 'a'] = i;
        }
        List<Integer> result = new ArrayList<>();
        int start = 0, end = 0;
        for(int i = 0; i < n; i++) {
            end = Math.max(end, last[s.charAt(i) - 'a']);
            if(i == end) {
                result.add(end - start + 1);
                start = end + 1;
            }
        }
        return result;
    }
}
```

