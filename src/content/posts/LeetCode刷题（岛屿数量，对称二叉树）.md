---
title: LeetCode刷题（岛屿数量，对称二叉树）
published: 2025-06-08
updated: 2025-06-08
description: 'Leet'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 岛屿数量

## 题目描述

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

**示例 1：**

```
输入：grid = [
  ["1","1","1","1","0"],
  ["1","1","0","1","0"],
  ["1","1","0","0","0"],
  ["0","0","0","0","0"]
]
输出：1
```

**示例 2：**

```
输入：grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
输出：3
```



## 题解

深度优先遍历，遍历二维数组的每一个元素，当为陆地的时候，用dfs遍历将相邻的1都置为0，使count++

```java
class Solution {
    public int numIslands(char[][] grid) {
        int count = 0;
        for(int i = 0; i < grid.length; i++) {
            for(int j = 0; j < grid[0].length; j++) {
                if(grid[i][j] == '1') {
                    dfs(grid, i, j);
                    count++;
                }
            }
        }
        return count;
    }

    private void dfs(char[][] grid, int i, int j) {
        if(i >= 0 && i < grid.length && j >= 0 && j < grid[0].length && grid[i][j] == '1') {
            grid[i][j] = '0';
            dfs(grid, i - 1, j);
            dfs(grid, i, j - 1);
            dfs(grid, i + 1, j);
            dfs(grid, i, j + 1);
        }
    }
}
```



# 对称二叉树

## 题目描述

给你一个二叉树的根节点 `root` ， 检查它是否轴对称。

 

**示例 1：**

![182](../images/182.png)

```
输入：root = [1,2,2,3,4,4,3]
输出：true
```

**示例 2：**

![183](../images/183.png)

```
输入：root = [1,2,2,null,3,null,3]
输出：false
```



## 题解

判断一棵树是否对成，需要判断左右孩子是否等价，其中check方法是检测左右两个孩子是否等价，等价条件为两个子节点值相同，且p.left节点等价于q.right，q.left 等价于 p.right

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        return check(root.left, root.right);
    }

    private boolean check(TreeNode p, TreeNode q) {
        if(p == null && q == null) {
            return true;
        }
        if(p == null || q == null) {
            return false;
        }
        boolean flag1 = p.val == q.val;
        boolean flag2 = check(p.left, q.right);
        boolean flag3 = check(p.right, q.left);
        return flag1 && flag2 && flag3;
    }
}
```

