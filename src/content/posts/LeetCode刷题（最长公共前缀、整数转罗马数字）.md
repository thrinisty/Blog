---
title: LeetCode刷题（最长公共前缀、整数转罗马数字）
published: 2025-10-26
updated: 2025-10-26
description: '最长公共前缀、整数转罗马数字'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 最长公共前缀

## 题目描述

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 `""`。

**示例 1：**

```
输入：strs = ["flower","flow","flight"]
输出："fl"
```

**示例 2：**

```
输入：strs = ["dog","racecar","car"]
输出：""
解释：输入不存在公共前缀。
```



## 题解

先遍历一遍单词数组，取出最短的长度，这是我们的最长前缀上限

之后从下表0开始到上界之间依次选取，比较该下表的字符是否相等，如果有不等的情况说明最长前缀不包含当前字符，提前返回前边的字符串

如果都遍历结束，代表最长前缀上界之前的元素都相等，返回上界前的部分

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        int n = strs.length;
        int length = Integer.MAX_VALUE;
        for(int i = 0; i < n; i++) {
            length = Math.min(length, strs[i].length());
        }
        for(int i = 0; i < length; i++) {
            for(int j = 0; j < n - 1; j++) {
                if(strs[j].charAt(i) != strs[j + 1].charAt(i)) {
                    return strs[0].substring(0, i);
                }
            }
        }
        return strs[0].substring(0, length);
    }
}
```



# 整数转罗马数字

## 题目描述

七个不同的符号代表罗马数字，其值如下：

| 符号 | 值   |
| ---- | ---- |
| I    | 1    |
| V    | 5    |
| X    | 10   |
| L    | 50   |
| C    | 100  |
| D    | 500  |
| M    | 1000 |

罗马数字是通过添加从最高到最低的小数位值的转换而形成的。将小数位值转换为罗马数字有以下规则：

- 如果该值不是以 4 或 9 开头，请选择可以从输入中减去的最大值的符号，将该符号附加到结果，减去其值，然后将其余部分转换为罗马数字。
- 如果该值以 4 或 9 开头，使用 **减法形式**，表示从以下符号中减去一个符号，例如 4 是 5 (`V`) 减 1 (`I`): `IV` ，9 是 10 (`X`) 减 1 (`I`)：`IX`。仅使用以下减法形式：4 (`IV`)，9 (`IX`)，40 (`XL`)，90 (`XC`)，400 (`CD`) 和 900 (`CM`)。
- 只有 10 的次方（`I`, `X`, `C`, `M`）最多可以连续附加 3 次以代表 10 的倍数。你不能多次附加 5 (`V`)，50 (`L`) 或 500 (`D`)。如果需要将符号附加4次，请使用 **减法形式**。

给定一个整数，将其转换为罗马数字。

**示例 1：**

**输入：**num = 3749

**输出：** "MMMDCCXLIX"

**解释：**

```
3000 = MMM 由于 1000 (M) + 1000 (M) + 1000 (M)
 700 = DCC 由于 500 (D) + 100 (C) + 100 (C)
  40 = XL 由于 50 (L) 减 10 (X)
   9 = IX 由于 10 (X) 减 1 (I)
注意：49 不是 50 (L) 减 1 (I) 因为转换是基于小数位
```

**示例 2：**

**输入：**num = 58

**输出：**"LVIII"

**解释：**

```
50 = L
 8 = VIII
```

**示例 3：**

**输入：**num = 1994

**输出：**"MCMXCIV"

**解释：**

```
1000 = M
 900 = CM
  90 = XC
   4 = IV
```



## 题解

将字符表放入数组中，用贪心算法从高向低依次选取符合条件的字符，添加到目标字符中，并同时减去字符代表的数字，直到num为0为止

```java
class Solution {
    public String intToRoman(int num) {
        int[] values = {1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1};
        String[] symbols = {"M", "CM", "D", "CD", "C", "XC", "L", "XL", "X", "IX", "V", "IV", "I"};
        StringBuilder str = new StringBuilder();
        for(int i = 0; i < values.length && num > 0; i++) {
            while(num >= values[i]) {
                str.append(symbols[i]);
                num -= values[i];
            }
        } 
        return str.toString();
    }
}
```

