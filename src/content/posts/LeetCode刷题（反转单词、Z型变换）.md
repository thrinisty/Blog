---
title: LeetCode刷题（反转单词、Z型变换）
published: 2025-10-27
updated: 2025-10-27
description: '反转字符串中的单词、Z字形变换'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 反转字符串中的单词

## 题目描述

给你一个字符串 `s` ，请你反转字符串中 **单词** 的顺序。

**单词** 是由非空格字符组成的字符串。`s` 中使用至少一个空格将字符串中的 **单词** 分隔开。

返回 **单词** 顺序颠倒且 **单词** 之间用单个空格连接的结果字符串。

**注意：**输入字符串 `s`中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。

**示例 1：**

```
输入：s = "the sky is blue"
输出："blue is sky the"
```

**示例 2：**

```
输入：s = "  hello world  "
输出："world hello"
解释：反转后的字符串中不能存在前导空格和尾随空格。
```

**示例 3：**

```
输入：s = "a good   example"
输出："example good a"
解释：如果两个单词间有多余的空格，反转后的字符串需要将单词间的空格减少到仅有一个。
```



## 题解

通过左右指针每轮指定单词的左右边界，将单词放入list集合备用，最后将集合中的元素逐个添加至结果中

```java
class Solution {
    public String reverseWords(String s) {
        List<String> list = new ArrayList<>();
        int n = s.length();
        int left = 0;
        int right = 0;
        while(right < n) {
            while(right < n && s.charAt(right) == ' ') {
                right++;
            }
            left = right;
            while(right < n && s.charAt(right) != ' ') {
                right++;
            }
            //这里的作用是防止最后空格字符被加入到单词中
            if(left < n) {
                list.add(s.substring(left, right));
            }
        }
        StringBuilder result = new StringBuilder();
        for(int i = list.size() - 1; i >= 1; i--) {
            result.append(list.get(i));
            result.append(" ");
        } 
        result.append(list.get(0));
        return result.toString();
    }
}
```



# Z字形变换

## 题目描述

将一个给定字符串 `s` 根据给定的行数 `numRows` ，以从上往下、从左到右进行 Z 字形排列。

比如输入字符串为 `"PAYPALISHIRING"` 行数为 `3` 时，排列如下：

```
P   A   H   N
A P L S I I G
Y   I   R
```

之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如：`"PAHNAPLSIIGYIR"`。

请你实现这个将字符串进行指定行数变换的函数：

```
string convert(string s, int numRows);
```

**示例 1：**

```
输入：s = "PAYPALISHIRING", numRows = 3
输出："PAHNAPLSIIGYIR"
```

**示例 2：**

```
输入：s = "PAYPALISHIRING", numRows = 4
输出："PINALSIGYAHRPI"
解释：
P     I    N
A   L S  I G
Y A   H R
P     I
```

**示例 3：**

```
输入：s = "A", numRows = 1
输出："A"
```



## 题解

设立一个List，存储每一行的字符，逐个遍历char c，设立i标识当前字符应该放入到的行数，当i为第一行或者最后一行时，将flag反转，最后下一个位置应该是i+flag的值

```java
class Solution {
    public String convert(String s, int numRows) {
        if(numRows < 2) {
            return s;
        }
        List<StringBuilder> rows = new ArrayList<>();
        for(int i = 0; i < numRows; i++) {
            rows.add(new StringBuilder());
        }
        int i = 0;
        int flag = -1;
        for(char c : s.toCharArray()) {
            rows.get(i).append(c);
            if(i == 0 || i == numRows - 1) {
                flag = -flag;
            }
            i += flag;
        }
        StringBuilder result = new StringBuilder();
        for(StringBuilder str : rows) {
            result.append(str);
        }
        return result.toString();
    }
}
```



