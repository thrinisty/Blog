---
title: LeetCode刷题（只出现一次的数字Ⅱ、数字范围按位与）
published: 2025-11-25
updated: 2025-11-25
description: '只出现一次的数字Ⅱ、数字范围按位与'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 只出现一次的数字Ⅱ

## 题目描述

给你一个整数数组 `nums` ，除某个元素仅出现 **一次** 外，其余每个元素都恰出现 **三次 。**请你找出并返回那个只出现了一次的元素。

你必须设计并实现线性时间复杂度的算法且使用常数级空间来解决此问题。

 

**示例 1：**

```
输入：nums = [2,2,3,2]
输出：3
```

**示例 2：**

```
输入：nums = [0,1,0,1,0,1,99]
输出：99
```

## 题解

解法一：HashMap计数

```java
class Solution {
    public int singleNumber(int[] nums) {
        Map<Integer, Integer> map = new HashMap<>();
        for(int num : nums) {
            int count = map.getOrDefault(num, 0);
            map.put(num, count + 1);
        }
        Set<Integer> keySet = map.keySet();
        int target = -1;
        for(Integer key : keySet) {
            int count = map.get(key);
            if(count == 1) {
                return key;
            }
        }
        return target;
    }
}
```

解法二：位运算

这个部分最好联系离散数学的表达式写一个状态表进行求解

设当前状态为XY，输入为Z，那么我们可以分别为X和Y列出真值表

1	00	0	0	0
2	01	0	0	1
3	10	0	1	0
4	00	1	0	1
5	01	1	1	0
6	10	1	0	0

Y new = ~X ⋅ Y ⋅~Z+ ~X ⋅~Y⋅ Z

X同理也可以列出逻辑表达式

```java
class Solution {
    public int singleNumber(int[] nums) {
        int a = 0, b = 0;
        for(int num : nums) {
            a = (a ^ num) & ~b;
            b = (b ^ num) & ~a;
        }
        return a;
    }
}
```

# 数字范围按位与

## 题目描述

给你两个整数 `left` 和 `right` ，表示区间 `[left, right]` ，返回此区间内所有数字 **按位与** 的结果（包含 `left` 、`right` 端点）。

**示例 1：**

```
输入：left = 5, right = 7
输出：4
```

**示例 2：**

```
输入：left = 0, right = 0
输出：0
```

**示例 3：**

```
输入：left = 1, right = 2147483647
输出：0
```



## 题解

1. **按位与的性质**：
   - 任何数字与0按位与结果为0
   - 只有所有数字在某一位都为1时，结果的这一位才会是1
2. **关键观察**：
   - 区间内所有数字的按位与结果其实就是这些数字的二进制表示的公共前缀
   - 随着数字增加，最低有效位（最右边的1）会不断变化
3. **算法操作**：
   - `right &= right - 1` 这个操作会将 `right` 的最右边的1变为0
   - 重复这个过程，直到 `right` 小于 `left`，此时剩下的就是公共前缀

```java
class Solution {
    public int rangeBitwiseAnd(int left, int right) {
        while(left < right) {
            right &= right - 1;
        }
        return right;
    }
}
```

