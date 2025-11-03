---
title: LeetCode刷题（螺旋矩阵、矩阵置零）
published: 2025-10-31
updated: 2025-10-31
description: '螺旋矩阵、矩阵置零'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 螺旋矩阵

## 题目描述

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

 

**示例 1：**

![335](../images/335.jpg)

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[1,2,3,6,9,8,7,4,5]
```

**示例 2：**

![336](../images/336.jpg)

```
输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
输出：[1,2,3,4,8,12,11,10,9,5,6,7]
```



## 题解

通过设置四个边界，依次循环四条边界并迭代边界，直至边界冲突

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        int row = matrix.length;
        int col = matrix[0].length;
        List<Integer> result = new ArrayList<>();
        int left = 0;
        int right = col - 1;
        int up = 0;
        int down = row - 1;
        while(true) {
            for(int i = left; i <= right; i++) result.add(matrix[up][i]);
            if(++up > down) break;
            for(int i = up; i <= down; i++) result.add(matrix[i][right]);
            if(left > --right) break;
            for(int i = right; i >= left; i--) result.add(matrix[down][i]);
            if(up > --down) break;
            for(int i = down; i >= up; i--) result.add(matrix[i][left]);
            if(++left > right) break;
        }
        return result;
    }
}
```

# 矩阵置零

## 题目描述

给定一个 `*m* x *n*` 的矩阵，如果一个元素为 **0** ，则将其所在行和列的所有元素都设为 **0** 。请使用 **[原地](http://baike.baidu.com/item/原地算法)** 算法**。**

**示例 1：**

![337](../images/337.jpg)

```
输入：matrix = [[1,1,1],[1,0,1],[1,1,1]]
输出：[[1,0,1],[0,0,0],[1,0,1]]
```

**示例 2：**

![338](../images/338.jpg)

```
输入：matrix = [[0,1,2,0],[3,4,5,2],[1,3,1,5]]
输出：[[0,0,0,0],[0,4,5,0],[0,3,1,0]]
```



## 题解

使用行Set和列Set存储需要清零的行列，将两个Set中的对应行/列置为0，空间消耗为O(m + n)

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        Set<Integer> zeroRow = new HashSet<>();
        Set<Integer> zeroCol = new HashSet<>();
        int row = matrix.length;
        int col = matrix[0].length;
        for(int i = 0; i < row; i++) {
            for(int j = 0; j < col; j++) {
                if(matrix[i][j] == 0) {
                    zeroRow.add(i);
                    zeroCol.add(j);
                }
            }
        }
        for(Integer r : zeroRow) {
            for(int j = 0; j < col; j++) {
                matrix[r][j] = 0;
            }
        }
        for(Integer c : zeroCol) {
            for(int i = 0; i < row; i++) {
                matrix[i][c] = 0;
            }
        }
    }
}
```

