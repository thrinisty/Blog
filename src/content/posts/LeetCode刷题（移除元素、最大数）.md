---
title: LeetCode刷题（移除元素、最大数）
published: 2025-10-22
updated: 2025-10-22
description: '今天同步刷top150面试题，每日两道，剩余的hop100的10题看情况完成，有重复的直接跳过不计入每日两道题目，除此以外看情况完成面试实战题目'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 移除元素

## 题目描述

给你一个数组 `nums` 和一个值 `val`，你需要 **[原地](https://baike.baidu.com/item/原地算法)** 移除所有数值等于 `val` 的元素。元素的顺序可能发生改变。然后返回 `nums` 中与 `val` 不同的元素的数量。

假设 `nums` 中不等于 `val` 的元素数量为 `k`，要通过此题，您需要执行以下操作：

- 更改 `nums` 数组，使 `nums` 的前 `k` 个元素包含不等于 `val` 的元素。`nums` 的其余元素和 `nums` 的大小并不重要。
- 返回 `k`。

**示例 1：**

```
输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2,_,_]
解释：你的函数函数应该返回 k = 2, 并且 nums 中的前两个元素均为 2。
你在返回的 k 个元素之外留下了什么并不重要（因此它们并不计入评测）。
```

**示例 2：**

```
输入：nums = [0,1,2,2,3,0,4,2], val = 2
输出：5, nums = [0,1,4,0,3,_,_,_]
解释：你的函数应该返回 k = 5，并且 nums 中的前五个元素为 0,0,1,3,4。
注意这五个元素可以任意顺序返回。
你在返回的 k 个元素之外留下了什么并不重要（因此它们并不计入评测）。
```



## 题解

利用双指针解决

```java
class Solution {
    public int removeElement(int[] nums, int val) {
        int result = 0;
        int n = nums.length;
        for(int i = 0; i < n; i++) {
            if(nums[i] != val) {
                nums[result] = nums[i];
                result++;
            }
        }
        return result;
    }
}
```



# 最大数

## 题目描述

给定一组非负整数 `nums`，重新排列每个数的顺序（每个数不可拆分）使之组成一个最大的整数。

**注意：**输出结果可能非常大，所以你需要返回一个字符串而不是整数。

**示例 1：**

```
输入：nums = [10,2]
输出："210"
```

**示例 2：**

```
输入：nums = [3,30,34,5,9]
输出："9534330"
```



## 题解

将int数组转化为String类型，利用字符串自带的比较方法进行从大到小排序，最后拼接为字符串即可

```java
class Solution {
    public String largestNumber(int[] nums) {
        String[] strs = new String[nums.length];
        for(int i = 0; i < nums.length; i++) {
            strs[i] = nums[i] + "";
        }
        Arrays.sort(strs, (x, y) -> (y + x).compareTo(x + y));
        if(strs[0].equals("0")) {
            return "0";
        }
        StringBuilder result = new StringBuilder();
        for(String str : strs) {
            result.append(str);
        }
        return result.toString();
    }
}
```

附带一个冒泡排序的方法

```java
class Solution {
    public String largestNumber(int[] nums) {
        String[] strs = new String[nums.length];
        for(int i = 0; i < nums.length; i++) {
            strs[i] = nums[i] + "";
        }
        // Arrays.sort(strs, (x, y) -> (y + x).compareTo(x + y));
        int n = nums.length;
        for(int i = 0; i < n - 1; i++) {
            for(int j = 0; j < n - i - 1; j++) {
                if((strs[j] + strs[j + 1]).compareTo(strs[j + 1] + strs[j]) < 0) {
                    String temp = strs[j];
                    strs[j] = strs[j + 1];
                    strs[j + 1] = temp;
                }
            }
        }
        if(strs[0].equals("0")) {
            return "0";
        }
        StringBuilder result = new StringBuilder();
        for(String str : strs) {
            result.append(str);
        }
        return result.toString();
    }
}
```

