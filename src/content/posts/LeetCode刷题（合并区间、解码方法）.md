---
title: LeetCode刷题（合并区间、解码方法）
published: 2025-10-21
updated: 2025-10-21
description: ''
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 合并区间

## 题目描述

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。

**示例 1：**

```
输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
```

**示例 2：**

```
输入：intervals = [[1,4],[4,5]]
输出：[[1,5]]
解释：区间 [1,4] 和 [4,5] 可被视为重叠区间。
```

**示例 3：**

```
输入：intervals = [[4,7],[1,4]]
输出：[[1,7]]
解释：区间 [1,4] 和 [4,7] 可被视为重叠区间。
```

## 题解

先进行排序，排序后构造List集合依次遍历所有区间，对于List为空和后区间的小值大与前区间的情况直接将区间放入，否则更新最近的右边界，取值为取出的右边界和遍历右边界最大数值

最后将List转化为int[][]数组

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        Arrays.sort(intervals, (n1, n2) -> {
            return n1[0] - n2[0];
        });
        List<int[]> list = new ArrayList<>();
        for(int[] range : intervals) {
            if(list.isEmpty() || list.get(list.size() - 1)[1] < range[0]) {
                list.add(range);
            } else {
                int[] target = list.get(list.size() - 1);
                target[1] = Math.max(target[1], range[1]);
            }
        }
        int[][] result = new int[list.size()][2];
        for(int i = 0; i < result.length; i++) {
            result[i] = list.get(i);
        }
        return result;
    }
}
```



# 解码方法

## 题目描述

一条包含字母 `A-Z` 的消息通过以下映射进行了 **编码** ：

```
"1" -> 'A' "2" -> 'B' ... "25" -> 'Y' "26" -> 'Z'
```

然而，在 **解码** 已编码的消息时，你意识到有许多不同的方式来解码，因为有些编码被包含在其它编码当中（`"2"` 和 `"5"` 与 `"25"`）。

例如，`"11106"` 可以映射为：

- `"AAJF"` ，将消息分组为 `(1, 1, 10, 6)`
- `"KJF"` ，将消息分组为 `(11, 10, 6)`
- 消息不能分组为 `(1, 11, 06)` ，因为 `"06"` 不是一个合法编码（只有 "6" 是合法的）。

注意，可能存在无法解码的字符串。

给你一个只含数字的 **非空** 字符串 `s` ，请计算并返回 **解码** 方法的 **总数** 。如果没有合法的方式解码整个字符串，返回 `0`。

题目数据保证答案肯定是一个 **32 位** 的整数。



## 题解

利用动态规划求解：创建dp数组，其中存储到第i个数解码的可能，单独的数字只要不是0，就可以把对应的可能加入到当前dp中，如果是两个数字的，转化为数字判断是否在10-26之间，是的话可以将dp向前推两位的可能加入到dp中

```java
class Solution {
    public int numDecodings(String s) {
        if(s.isEmpty() || s.charAt(0) == '0') {
            return 0;
        }
        int n = s.length();
        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;
        for(int i = 2; i <= n; i++) {
            if(s.charAt(i - 1) != '0') {
                dp[i] += dp[i - 1];
            }
            int num = Integer.parseInt(s.substring(i - 2, i));
            if(num >= 10 && num <= 26) {
                dp[i] += dp[i - 2];
            }
        }
        return dp[n];
    }
}
```

