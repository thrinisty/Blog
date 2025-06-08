---
title: LeetCode刷题（编辑距离，颜色分类）
published: 2025-06-07
updated: 2025-06-07
description: '编辑距离，荷兰国旗问题'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 编辑距离

## 题目描述

给你两个单词 `word1` 和 `word2`， *请返回将 `word1` 转换成 `word2` 所使用的最少操作数* 。

你可以对一个单词进行如下三种操作：

- 插入一个字符
- 删除一个字符
- 替换一个字符

**示例 1：**

```
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```

**示例 2：**

```
输入：word1 = "intention", word2 = "execution"
输出：5
解释：
intention -> inention (删除 't')
inention -> enention (将 'i' 替换为 'e')
enention -> exention (将 'n' 替换为 'x')
exention -> exection (将 'n' 替换为 'c')
exection -> execution (插入 'u')
```



## 题解

设立一个dp二维数组，存放word1与word2的前n个字符需要操作的情况，初始的情况是两个都没有字符，赋值为0，分别右下遍历二维数组，代表着从空字符串开始需要增加操作字符的数量迭代加一

二维遍历整个数组，分为两种情况：

1.即将放入的两个字符相等，直接从左上方的第一个二维数组元素取值即可，代表着不需要操作

2.当字符不同的时候，可以从两个字符的左上情况进行替换操作，或者从左或者上进行增删操作，取三种方式的最小值，将取得的结果加一代表经历了一种操作

最后对于遍历的元素赋予对应的数值

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int n1 = word1.length();
        int n2 = word2.length();
        int[][] dp = new int[n1 + 1][n2 + 1];
        for(int i = 1; i <= n1; i++) {
            dp[i][0] = dp[i - 1][0] + 1;
        }
        for(int j = 1; j <= n2; j++) {
            dp[0][j] = dp[0][j - 1] + 1;
        }
        for(int i = 1; i <= n1; i++) {
            for(int j = 1; j <= n2; j++) {
                if(word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    int min = Math.min(dp[i - 1][j - 1], dp[i - 1][j]);
                    min = Math.min(min, dp[i][j - 1]);
                    dp[i][j] = min + 1;
                }
            }
        }
        return dp[n1][n2];
    }
}
```



# 颜色分类

## 题目描述

给定一个包含红色、白色和蓝色、共 `n` 个元素的数组 `nums` ，**[原地](https://baike.baidu.com/item/原地算法)** 对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

我们使用整数 `0`、 `1` 和 `2` 分别表示红色、白色和蓝色。

必须在不使用库内置的 sort 函数的情况下解决这个问题。



**示例 1：**

```
输入：nums = [2,0,2,1,1,0]
输出：[0,0,1,1,2,2]
```

**示例 2：**

```
输入：nums = [2,0,1]
输出：[0,1,2]
```



## 题解

解法一：反骨一下用一个临时的数组存储一下，把0方法到最前面，2放到最后面，剩余的用1填充

```java
class Solution {
    public void sortColors(int[] nums) {
        int n = nums.length;
        int[] result = new int[n];
        int min = 0;
        int max = n - 1;
        for(int i = 0; i < n; i++) {
            if(nums[i] == 0) {
                result[min++] = 0;
            } else if(nums[i] == 2) {
                result[max--] = 2;
            }
        }
        for(int i = min; i <= max; i++) {
            result[i] = 1;
        }
        for(int i = 0; i < n; i++) {
            nums[i] = result[i];
        }
    }
}
```



解法二：

用三个指针标识，flag位操作数指针，low之前的数为0，high之后的数为2

flag初始为0，当flag > high的时候表示处理完毕

当为0的时候需要和low指向的数字就行交换将0放入low的位置，low++，而flag因为之前的数字都处理过了所以flag++

当为1的时候，nums[flag]位置的数正确，flag++即可，无需交换放置

当为2的时候，和max处的位置进行交换，因为交换回的数字可能是2，max--，而flag不作调整，再次判断

```java
class Solution {
    public void sortColors(int[] nums) {
        int low = 0;
        int high = nums.length - 1;
        int flag = 0;
        while(flag <= high) {
            if(nums[flag] == 0) {
                swap(nums, low, flag);
                low++;
                flag++;
            } else if(nums[flag] == 1) {
                flag++;
            } else {
                swap(nums, flag, high);
                high--;
            }
        }
    }
    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

