---
title: LeetCode刷题（柱状图中的最大矩形、最小覆盖字串）
published: 2025-10-13
updated: 2025-10-13
description: '柱状图中的最大矩形（暴力超时）、最小覆盖字串'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 柱状图中的最大矩形

## 题目描述

给定 *n* 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。

求在该柱状图中，能够勾勒出来的矩形的最大面积。

**示例 1:**

![281](../images/281.jpg)

```
输入：heights = [2,1,5,6,2,3]
输出：10
解释：最大的矩形为图中红色区域，面积为 10
```

**示例 2：**

![282](../images/282.jpg)

```
输入： heights = [2,4]
输出： 4
```



## 题解

解法一：暴力枚举（93/99），对于每个数组元素取其高度，向左向右看其对应的的最长长度，用得到的面积迭代result

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int n = heights.length;
        int result = 0;
        for(int i = 0; i < n; i++) {
            int height = heights[i];
            int left = i;
            while(left - 1 >= 0 && heights[left - 1] >= height) {
                left--;
            }
            int right = i;
            while(right + 1 < n && heights[right + 1] >= height) {
                right++;
            }
            result = Math.max(result, height * (right - left + 1));
        }
        return result;
    }
}
```



# 最小覆盖字串

## 题目描述

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。

**注意：**

- 对于 `t` 中重复字符，我们寻找的子字符串中该字符数量必须不少于 `t` 中该字符数量。
- 如果 `s` 中存在这样的子串，我们保证它是唯一的答案。

**示例 1：**

```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
解释：最小覆盖子串 "BANC" 包含来自字符串 t 的 'A'、'B' 和 'C'。
```

**示例 2：**

```
输入：s = "a", t = "a"
输出："a"
解释：整个字符串 s 是最小覆盖子串。
```

**示例 3:**

```
输入: s = "a", t = "aa"
输出: ""
解释: t 中两个字符 'a' 均应包含在 s 的子串中，
因此没有符合条件的子字符串，返回空字符串。
```



## 题解

构建一个Map集合存储第二个字符需要匹配的字符类型count数量以及字符其所需要的数量，初始化答案的左边界以及右边界（这里合理定义就好，后续不发生改变则证明没有，匹配一下初始值返回空字符），构造一个滑动窗口左右边界分别为l和r，都初始化为0

用右边界遍历s字符串，如果当前字符在Map中使其数量减一，如果减完之后为0代表该字符匹配完成满足对应数量

当count为0的时候代表所有目标字符数量满足要求，这个时候是我们覆盖的一种情况，如果左右边界详见结果小于原先的结果就进行迭代，之后从左向右移动左边界，当当前的左边界的字符存在于map中将其对应需求数量加一，如果数量>0则count++，直到count不在等于0的时候说明当前情况不满足覆盖，进入下下一次循环，重复向后遍历右边界

```java
class Solution {
    public String minWindow(String s, String t) {
        Map<Character, Integer> map = new HashMap<>();
        for(char c : t.toCharArray()) {
            map.put(c, map.getOrDefault(c, 0) + 1);
        }
        int count = map.size();
        //这里的初始化包围s就好，注意最后的匹配返回""
        int result_l = -1;
        int result_r = s.length();
        for(int l = 0, r = 0; r < s.length(); r++) {
            char c = s.charAt(r);
            if(map.containsKey(c)) {
                int num = map.get(c);
                if(num == 1) {
                    count--;
                }
                map.put(c, num - 1);
            }
            while(count == 0) {
                if(result_r - result_l > r - l) {
                    result_l = l;
                    result_r = r;
                }
                char ch = s.charAt(l);
                if(map.containsKey(ch)) {
                    int num = map.get(ch);
                    if(num == 0) {
                        count++;
                    }
                    map.put(ch, num + 1);
                }
                l++;
            }
        }
        if(result_l == -1) return "";
        return s.substring(result_l, result_r + 1);
    }
}
```

