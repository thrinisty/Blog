---
title: LeetCode刷题（买股票，前k个高频元素）
published: 2025-06-04
updated: 2025-06-04
description: '买股票含冷冻期，前k个高频元素'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 买股票

## 题目描述

给定一个整数数组`prices`，其中第 `prices[i]` 表示第 `*i*` 天的股票价格 。

设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:

- 卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。

**注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1:**

```
输入: prices = [1,2,3,0,2]
输出: 3 
解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]
```

**示例 2:**

```
输入: prices = [1]
输出: 0
```



## 题解

用两个数组存储分别存储持有股票时的最大资产，以及售出股票的最大资产，在第一天将buy设置为-prices[0]

每天迭代数组的对应元素的时候，对于持有的数组元素取前一天的（持有股票资产）和（非持有股票买入当天股票后的资产）中的最大值，而对于售出股票的数组而言取（前一天非持有股票的资产）和（以往持有股票在当天售出）的最大值

:::tip

注意在买入股票的时候要求前一天不卖出，这一步的处理是迭代持有股票的数组的时候，用i-2处的非持有元素迭代（注意数组越界的处理，在i>=2的时候取0）

:::

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;
        int[] buy = new int[n];
        int[] sale = new int[n];
        buy[0] = -prices[0];
        for(int i = 1; i < n; i++) {
            buy[i] = Math.max(buy[i - 1], (i == 1 ?  0 : sale[i - 2]) - prices[i]);
            sale[i] = Math.max(sale[i - 1], buy[i - 1] + prices[i]);
        }
        return sale[n - 1];
    }
}
```



# 前k个高频元素

## 题目描述

给你一个整数数组 `nums` 和一个整数 `k` ，请你返回其中出现频率前 `k` 高的元素。你可以按 **任意顺序** 返回答案。

 

**示例 1:**

```
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
```

**示例 2:**

```
输入: nums = [1], k = 1
输出: [1]
```

**提示：**

- `1 <= nums.length <= 105`
- `k` 的取值范围是 `[1, 数组中不相同的元素的个数]`
- 题目数据保证答案唯一，换句话说，数组中前 `k` 个高频元素的集合是唯一的



## 题解

这一题我按照哈希桶的方式自己用数组的形式，将放入哈希表中的数字-出现次数，按照出现的次数作为下标放入了一个数组中

本地测评没有问题，但是后续验证有两个数字出现频率相同的情况下，只会有一个数字放入到最后的结果中，应该是相同的频率下，后放入的数字会覆盖掉之前放入的数字，更改为List集合的数组后成功通过

在处理List集合的时候从后向前处理，当List集合为空表示没有数字落在对应的频率中，continue继续，到有元素的List时迭代的将其中的元素放入结果集中

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        int[] result = new int[k];
        Map<Integer, Integer> map = new HashMap<>();
        for(int num : nums) {
            int i = map.getOrDefault(num, 0);
            map.put(num, i + 1);
        }

        List<Integer>[] list = new List[nums.length + 1];
        Set<Integer> set = map.keySet();
        for(int num : set) {
            int i = map.get(num);
            if(list[i] == null) {
                list[i] = new ArrayList<Integer>();
            }
            list[i].add(num);
        }

        int len = 0;
        for(int i = list.length - 1; i >= 0; i--) {
            if(list[i] == null) {
                continue;
            }
            for(int num : list[i]) {
                result[len] = num;
                len++;
                if(len == k) {
                    return result;
                }
            }
        }
        return result;
    }
}
```

