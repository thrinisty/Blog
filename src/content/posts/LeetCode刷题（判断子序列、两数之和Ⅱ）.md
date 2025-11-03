---
title: LeetCode刷题（判断子序列、两数之和Ⅱ）
published: 2025-10-29
updated: 2025-10-29
description: '判断子序列、两数之和Ⅱ'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 判断子序列

## 题目描述

给定字符串 **s** 和 **t** ，判断 **s** 是否为 **t** 的子序列。

字符串的一个子序列是原始字符串删除一些（也可以不删除）字符而不改变剩余字符相对位置形成的新字符串。（例如，`"ace"`是`"abcde"`的一个子序列，而`"aec"`不是）。

**进阶：**

如果有大量输入的 S，称作 S1, S2, ... , Sk 其中 k >= 10亿，你需要依次检查它们是否为 T 的子序列。在这种情况下，你会怎样改变代码？

**示例 1：**

```
输入：s = "abc", t = "ahbgdc"
输出：true
```

**示例 2：**

```
输入：s = "axc", t = "ahbgdc"
输出：false
```



## 题解

两个指针，依次遍历，有匹配元素则被匹配元素指针++，最后判断是否完全匹配

```java
class Solution {
    public boolean isSubsequence(String s, String t) {
        int n1 = s.length();
        int n2 = t.length();
        int i = 0;
        int j = 0;
        while(i < n1 && j < n2) {
            char c1 = s.charAt(i);
            char c2 = t.charAt(j);
            if(c1 == c2) {
                i++;
            }
            j++;
        }
        return i == n1;
    }
}
```



# 两数之和

## 题目描述

给你一个下标从 **1** 开始的整数数组 `numbers` ，该数组已按 **非递减顺序排列** ，请你从数组中找出满足相加之和等于目标数 `target` 的两个数。如果设这两个数分别是 `numbers[index1]` 和 `numbers[index2]` ，则 `1 <= index1 < index2 <= numbers.length` 。

以长度为 2 的整数数组 `[index1, index2]` 的形式返回这两个整数的下标 `index1` 和 `index2`。

你可以假设每个输入 **只对应唯一的答案** ，而且你 **不可以** 重复使用相同的元素。

你所设计的解决方案必须只使用常量级的额外空间。

**示例 1：**

```
输入：numbers = [2,7,11,15], target = 9
输出：[1,2]
解释：2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。返回 [1, 2] 。
```

**示例 2：**

```
输入：numbers = [2,3,4], target = 6
输出：[1,3]
解释：2 与 4 之和等于目标数 6 。因此 index1 = 1, index2 = 3 。返回 [1, 3] 。
```

**示例 3：**

```
输入：numbers = [-1,0], target = -1
输出：[1,2]
解释：-1 与 0 之和等于目标数 -1 。因此 index1 = 1, index2 = 2 。返回 [1, 2] 。
```



## 题解

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int left = 0;
        int right = numbers.length - 1;
        while(left < right) {
            int count = numbers[left] + numbers[right];
            if (count == target) {
                return new int[]{left + 1, right + 1};
            } else if (count < 0) {
                left++;
            } else {
                right--;
            }
        }
        return new int[2];
    }
}
```

