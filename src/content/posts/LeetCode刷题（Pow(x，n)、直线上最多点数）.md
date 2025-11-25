---
title: LeetCode刷题（Pow(x，n)、直线上最多点数）
published: 2025-11-23
updated: 2025-11-23
description: 'Pow(x，n)、直线上最多点数'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# Pow(x，n)

## 题目描述

实现 [pow(*x*, *n*)](https://www.cplusplus.com/reference/valarray/pow/) ，即计算 `x` 的整数 `n` 次幂函数（即，`xn` ）。

**示例 1：**

```
输入：x = 2.00000, n = 10
输出：1024.00000
```

**示例 2：**

```
输入：x = 2.10000, n = 3
输出：9.26100
```

**示例 3：**

```
输入：x = 2.00000, n = -2
输出：0.25000
解释：2-2 = 1/22 = 1/4 = 0.25
```

## 题解

解法一：依次×原数（超时）

```java
class Solution {
    public double myPow(double x, int n) {
        double result = 1;
        if(n == 0) {
            return 1;
        } else if(n > 0) {
            for(int i = 0; i < n; i++) {
                result *= x;
            }
        } else {
            n = -n;
            for(int i = 0; i < n; i++) {
                result *= 1.0 / x;
            }
        }
        return result;
    }
}
```

解法二：快速幂解析

每一次幂数末尾为1的时候让结果乘上原数，之后原数乘等于原数，最后右移幂数

```java
class Solution {
    public double myPow(double x, int n) {
        if(x == 0) return 0 ;
        double result = 1.0;
        long m = n;
        if(m < 0) {
            x = 1 / x;
            m = -m;
        }
        while(m > 0) {
            if((m & 1) == 1) result *= x;
            x *= x;
            m >>= 1;
        }
        return result;
    }
}
```



# 直线上最多点数

## 题目描述

给你一个数组 `points` ，其中 `points[i] = [xi, yi]` 表示 **X-Y** 平面上的一个点。求最多有多少个点在同一条直线上。

 

**示例 1：**

![389](../images/389.jpg)

```
输入：points = [[1,1],[2,2],[3,3]]
输出：3
```

**示例 2：**

![390](../images/390.jpg)

```
输入：points = [[1,1],[3,2],[5,3],[4,1],[2,3],[1,4]]
输出：4
```



## 题解

稍微在除法哪里卡了一下，后面发觉可以将除号转为乘号

```java
class Solution {
    public int maxPoints(int[][] points) {
        int n = points.length; 
        if(n <= 2) return n;
        int result = 0;
        for(int i = 0; i < n; i++) {
            int x1 = points[i][0];
            int y1 = points[i][1];
            for(int j = i + 1; j < n; j++) {
                int x2 = points[j][0];
                int y2 = points[j][1];
                int count = 0;
                for(int k = 0; k < n; k++) {
                    int x = points[k][0];
                    int y = points[k][1];
                    if((y - y1) * (x - x2) == (y - y2) * (x - x1)) {
                        count++;
                    }
                }
                result = Math.max(result, count);
            }
        }
        return result;
    }
}
```



