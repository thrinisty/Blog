---
title: LeetCode刷题（最大矩形）
published: 2025-12-12
updated: 2025-12-12
description: '最大矩形'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 最大矩形

## 题目描述

给定一个由 `0` 和 `1` 组成的矩阵 `matrix` ，找出只包含 `1` 的最大矩形，并返回其面积。

**注意：**此题 `matrix` 输入格式为一维 `01` 字符串数组。

**示例 1：**

![502](../images/502.jpg)

```
输入：matrix = ["10100","10111","11111","10010"]
输出：6
解释：最大矩形如上图所示。
```

**示例 2：**

```
输入：matrix = []
输出：0
```

**示例 3：**

```
输入：matrix = ["0"]
输出：0
```

**示例 4：**

```
输入：matrix = ["1"]
输出：1
```

**示例 5：**

```
输入：matrix = ["00"]
输出：0
```



## 题解

本质上和柱状图中的最大矩形思路一致，只不过现需要将图转化为row个一维数组，代表从该行出发向上的最大高度，用柱状图中的最大矩形的算法求每一行的最大矩形即可

```java
class Solution {
    public int maximalRectangle(String[] matrix) {
        if(matrix.length == 0) {
            return 0;
        }
        int row = matrix.length;
        int col = matrix[0].length();
        int[][] arr = new int[row][col];

        for(int i = 0; i < col; i++) {
            arr[0][i] = matrix[0].charAt(i) == '1' ? 1 : 0;
        }
        for(int i = 1; i < row; i++) {
            for(int j = 0; j < col; j++) {
                if(matrix[i].charAt(j) == '1') {
                    arr[i][j] = arr[i - 1][j] + 1;
                } else {
                    arr[i][j] = 0;
                }
            }
        }
        int result = 0;
        for(int i = 0; i < row; i++) {
            result = Math.max(result, maxArea(arr[i]));
        } 
        return result;
    }

    public int maxArea(int[] heights) {
        int n = heights.length;
        int[] temp = new int[n + 2];
        for(int i = 0; i < n; i++) {
            temp[i + 1] = heights[i];
        }
        Stack<Integer> stack = new Stack<>();
        stack.push(0);
        n = n + 2;
        int result = 0;
        for(int i = 1; i < n; i++) {
            while(temp[i] < temp[stack.peek()]) {
                int height = temp[stack.pop()];
                int length = i - 1 - stack.peek();
                result = Math.max(result, height * length);
            }
            stack.push(i);
        }
        return result;
    }
}
```

