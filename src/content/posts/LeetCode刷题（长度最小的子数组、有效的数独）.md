---
title: LeetCode刷题（长度最小的子数组、有效的数独）
published: 2025-10-30
updated: 2025-10-30
description: '长度最小的子数组、有效的数独'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 长度最小的子数组

## 题目描述

给定一个含有 `n` 个正整数的数组和一个正整数 `target` **。**

找出该数组中满足其总和大于等于 `target` 的长度最小的 **子数组** `[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度**。**如果不存在符合条件的子数组，返回 `0` 。

**示例 1：**

```
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

**示例 2：**

```
输入：target = 4, nums = [1,4,4]
输出：1
```

**示例 3：**

```
输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0
```

 

## 题解

建立双指针，右边的指针一直向右遍历，当和大于目标的时候循环移动左边的指针，当大于目标的时候迭代长度，知道不满足大于target

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int sum = 0;
        int result = Integer.MAX_VALUE;
        int left = 0;
        for(int right = 0; right < nums.length; right++) {
            sum += nums[right];
            if(sum >= target) {
                while(sum >= target) {
                    result = Math.min(result, right - left + 1);
                    sum -= nums[left++];
                }
            }
        }
        return result == Integer.MAX_VALUE ? 0 : result;
    }
}
```



# 有效的数独

## 题目描述

请你判断一个 `9 x 9` 的数独是否有效。只需要 **根据以下规则** ，验证已经填入的数字是否有效即可。

1. 数字 `1-9` 在每一行只能出现一次。
2. 数字 `1-9` 在每一列只能出现一次。
3. 数字 `1-9` 在每一个以粗实线分隔的 `3x3` 宫内只能出现一次。（请参考示例图）

 

**注意：**

- 一个有效的数独（部分已被填充）不一定是可解的。
- 只需要根据以上规则，验证已经填入的数字是否有效即可。
- 空白格用 `'.'` 表示。

 

**示例 1：**

![333](../images/333.png)

```
输入：board = 
[["5","3",".",".","7",".",".",".","."]
,["6",".",".","1","9","5",".",".","."]
,[".","9","8",".",".",".",".","6","."]
,["8",".",".",".","6",".",".",".","3"]
,["4",".",".","8",".","3",".",".","1"]
,["7",".",".",".","2",".",".",".","6"]
,[".","6",".",".",".",".","2","8","."]
,[".",".",".","4","1","9",".",".","5"]
,[".",".",".",".","8",".",".","7","9"]]
输出：true
```

**示例 2：**

```
输入：board = 
[["8","3",".",".","7",".",".",".","."]
,["6",".",".","1","9","5",".",".","."]
,[".","9","8",".",".",".",".","6","."]
,["8",".",".",".","6",".",".",".","3"]
,["4",".",".","8",".","3",".",".","1"]
,["7",".",".",".","2",".",".",".","6"]
,[".","6",".",".",".",".","2","8","."]
,[".",".",".","4","1","9",".",".","5"]
,[".",".",".",".","8",".",".","7","9"]]
输出：false
解释：除了第一行的第一个数字从 5 改为 8 以外，空格内其他数字均与 示例1 相同。 但由于位于左上角的 3x3 宫内有两个 8 存在, 因此这个数独是无效的。
```

## 题解

不知道还有没有其他的方法，我的解法比较复杂

```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        Set<Character> set = new HashSet<>();
        for (int i = 0; i < 9; i++) {
            set.clear();
            for (int j = 0; j < 9; j++) {
                char c = board[i][j];
                if (c != '.') {
                    if (set.contains(c)) {
                        return false;
                    }
                    set.add(c);
                }
            }
        }
        for (int j = 0; j < 9; j++) {
            set.clear();
            for (int i = 0; i < 9; i++) {
                char c = board[i][j];
                if (c != '.') {
                    if (set.contains(c)) {
                        return false;
                    }
                    set.add(c);
                }
            }
        }
        for (int cow_num = 0; cow_num < 3; cow_num++) {
            for (int rol_num = 0; rol_num < 3; rol_num++) {
                int i_idx = cow_num * 3;
                int j_idx = rol_num * 3;
                set.clear();
                for (int i = i_idx; i < i_idx + 3; i++) {
                    for (int j = j_idx; j < j_idx + 3; j++) {
                        char c = board[i][j];
                        if (c != '.') {
                            if (set.contains(c)) {
                                return false;
                            }
                            set.add(c);
                        }
                    }
                }
            }
        }
        return true;
    }
}
```

