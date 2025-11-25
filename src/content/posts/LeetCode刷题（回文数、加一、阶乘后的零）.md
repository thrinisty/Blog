---
title: LeetCode刷题（回文数、加一、阶乘后的零）
published: 2025-11-22
updated: 2025-11-22
description: '回文数、加一、阶乘后的零'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 回文数

## 题目描述

给你一个整数 `x` ，如果 `x` 是一个回文整数，返回 `true` ；否则，返回 `false` 。

回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

- 例如，`121` 是回文，而 `123` 不是。

**示例 1：**

```
输入：x = 121
输出：true
```

**示例 2：**

```
输入：x = -121
输出：false
解释：从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。
```

**示例 3：**

```
输入：x = 10
输出：false
解释：从右向左读, 为 01 。因此它不是一个回文数。
```



## 题解

对每一位取出，构造对应的回文数，最后判断回文数是否和原先的数相等

```java
class Solution {
    public boolean isPalindrome(int x) {
        if(x < 0) return false;
        int num = 0;
        int cur = x;
        while(cur != 0) {
            int low = cur % 10;
            num = num * 10 + low;
            cur /= 10;
        }
        return x == num;
    }
}
```



# 加一

## 题目描述

给定一个表示 **大整数** 的整数数组 `digits`，其中 `digits[i]` 是整数的第 `i` 位数字。这些数字按从左到右，从最高位到最低位排列。这个大整数不包含任何前导 `0`。

将大整数加 1，并返回结果的数字数组。

**示例 1：**

```
输入：digits = [1,2,3]
输出：[1,2,4]
解释：输入数组表示数字 123。
加 1 后得到 123 + 1 = 124。
因此，结果应该是 [1,2,4]。
```

**示例 2：**

```
输入：digits = [4,3,2,1]
输出：[4,3,2,2]
解释：输入数组表示数字 4321。
加 1 后得到 4321 + 1 = 4322。
因此，结果应该是 [4,3,2,2]。
```

**示例 3：**

```
输入：digits = [9]
输出：[1,0]
解释：输入数组表示数字 9。
加 1 得到了 9 + 1 = 10。
因此，结果应该是 [1,0]。
```



## 题解

先在末尾加1，之后依次判断是否是10，如果是则前位++，本位为0，最后第一位特判以下做复制处理

```java
class Solution {
    public int[] plusOne(int[] digits) {
        int n = digits.length;
        digits[n - 1] += 1;
        for(int i = n - 1; i >= 1; i--) {
            if(digits[i] == 10) {
                digits[i] = 0;
                digits[i - 1] += 1;
            }
        }
        if(digits[0] != 10) return digits;
        digits[0] = 0;
        int[] result = new int[n + 1];
        result[0] = 1;
        for(int i = 1; i <= n; i++) {
            result[i] = digits[i - 1];
        }
        return result;
    }
}
```



# 阶乘后的零

## 题目描述

给定一个整数 `n` ，返回 `n!` 结果中尾随零的数量。

提示 `n! = n * (n - 1) * (n - 2) * ... * 3 * 2 * 1`

**示例1：**

```
输入：n = 3
输出：0
解释：3! = 6 ，不含尾随 0
```

**示例 2：**

```
输入：n = 5
输出：1
解释：5! = 120 ，有一个尾随 0
```

**示例 3：**

```
输入：n = 0
输出：0
```



## 题解

解法一：计算阶乘之后计算0数（结果溢出导致错误）

```java
class Solution {
    public int trailingZeroes(int n) {
        int sum = 1;
        for(int i = 1; i <= n; i++) {
            sum *= i;
        }
        int result = 0;
        while(sum != 0) {
            int low = sum % 10;
            if(low == 0) {
                result++;
            } else {
                break;
            }
            sum /= 10;
        }
        return result;
    }
}
```

解法二：数学大法，what can I say? 背就完了

大概的思路是将0个数改为计算多少对（2、5），也就是5出现的数

```java
class Solution {
    public int trailingZeroes(int n) {
        int count = 0;
        while(n > 0) {
            count += n / 5;
            n /= 5;
        }
        return count;
    }
}
```

