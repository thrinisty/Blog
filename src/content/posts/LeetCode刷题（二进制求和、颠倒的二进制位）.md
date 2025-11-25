---
title: LeetCode刷题（二进制求和、颠倒的二进制位）
published: 2025-11-24
updated: 2025-11-24
description: '二进制求和、颠倒的二进制位'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 二进制求和

## 题目描述

给你两个二进制字符串 `a` 和 `b` ，以二进制字符串的形式返回它们的和。

 

**示例 1：**

```
输入:a = "11", b = "1"
输出："100"
```

**示例 2：**

```
输入：a = "1010", b = "1011"
输出："10101"
```

**提示：**

- `1 <= a.length, b.length <= 104`
- `a` 和 `b` 仅由字符 `'0'` 或 `'1'` 组成
- 字符串如果不是 `"0"` ，就不含前导零



## 题解

思路和两数之和类似，构造一个数组存储数据依次相加即可

```java
class Solution {
    public String addBinary(String a, String b) {
        int n1 = a.length();
        int n2 = b.length();
        int n = Math.max(n1, n2) + 1;
        int[] arr = new int[n];
        int p = n - 1;
        int i = n1 - 1;
        int j = n2 - 1;
        int count = 0;
        while(i >= 0 || j >= 0 || count != 0) {
            int num1 = i >= 0 ? Integer.parseInt(a.charAt(i) + "") : 0;
            int num2 = j >= 0 ? Integer.parseInt(b.charAt(j) + "") : 0;
            int sum = num1 + num2 + count;
            arr[p--] = sum % 2;
            count = sum / 2;
            if(i >= 0) i--;
            if(j >= 0) j--;
        }
        StringBuilder result = new StringBuilder();
        if(arr[0] == 1) result.append(1);
        for(int k = 1; k < n; k++) {
            result.append(arr[k]);
        }
        return result.toString();
    }
}
```



# 颠倒的二进制位

## 题目描述

颠倒给定的 32 位有符号整数的二进制位。

**示例 1：**

**输入：**n = 43261596

**输出：**964176192

**解释：**

| 整数      | 二进制                           |
| --------- | -------------------------------- |
| 43261596  | 00000010100101000001111010011100 |
| 964176192 | 00111001011110000010100101000000 |

**示例 2：**

**输入：**n = 2147483644

**输出：**1073741822

**解释：**

| 整数       | 二进制                           |
| ---------- | -------------------------------- |
| 2147483644 | 01111111111111111111111111111100 |
| 1073741822 | 00111111111111111111111111111110 |

 

**提示：**

- `0 <= n <= 231 - 2`
- `n` 为偶数

**进阶**: 如果多次调用这个函数，你将如何优化你的算法？



## 题解

讲n转化为二进制数据放入数组中，再从左向右构造最后的结果

```java
class Solution {
    public int reverseBits(int n) {
        int[] arr = new int[32];
        int i = 31;
        while(n > 0) {
            arr[i--] = n % 2;
            n /= 2;
        }
        int result = 0;
        int temp = 1;
        for(i = 0; i < 32; i++) {
            if(arr[i] == 1) {
                result += temp;
            }
            temp <<= 1;
        }
        return result;
    }
}
```

