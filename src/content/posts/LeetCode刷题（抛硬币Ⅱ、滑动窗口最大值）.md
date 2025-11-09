---
title: LeetCode刷题（抛硬币Ⅱ、滑动窗口最大值）
published: 2025-11-06
updated: 2025-11-06
description: '新凯莱面试题目'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 抛硬币Ⅱ

## 题目描述

给你一个整数数组 `coins` 表示不同面额的硬币，另给一个整数 `amount` 表示总金额。

请你计算并返回可以凑成总金额的硬币组合数。如果任何硬币组合都无法凑出总金额，返回 `0` 。

假设每一种面额的硬币有无限个。 

题目数据保证结果符合 32 位带符号整数。

**示例 1：**

```
输入：amount = 5, coins = [1, 2, 5]
输出：4
解释：有四种方式可以凑成总金额：
5=5
5=2+2+1
5=2+1+1+1
5=1+1+1+1+1
```

**示例 2：**

```
输入：amount = 3, coins = [2]
输出：0
解释：只用面额 2 的硬币不能凑成总金额 3 。
```

**示例 3：**

```
输入：amount = 10, coins = [10] 
输出：1
```



## 题解

笔试用的回溯方法做的，超时

```java
package com.num1;

import java.util.*;

public class Main {
    private static int result = 0;
    private static List<Integer> list = new ArrayList<Integer>();
    private static List<List<Integer>> listResult = new ArrayList<>();
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        String str = in.nextLine();
        String[] arr = str.split(" ");
        int[] nums = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            nums[i] = Integer.parseInt(arr[i]);
        }
        int target = in.nextInt();
        dfs(nums, target, 0, 0);
        System.out.println(result);
        for(List<Integer> list : listResult) {
            System.out.println(list);
        }
    }

    public static void dfs(int[] nums, int target, int count, int index) {
        if (index == nums.length) {
            return;
        }
        if (target == count) {
            result++;
            listResult.add(new ArrayList<>(list));
            return;
        }
        if (count + nums[index] <= target) {
            list.add(nums[index]);
            dfs(nums, target, count + nums[index], index);
            list.remove(list.size() - 1);
        }
        dfs(nums, target, count, index + 1);
    }
}
```

外层用硬币进行遍历，内层迭代每个取值的可能情况，以保证硬币不会重复

```java
class Solution {
    public int change(int amount, int[] coins) {
        int n = coins.length;
        int[] dp = new int[amount + 1];
        dp[0] = 1;
        for(int coin : coins) {
            for(int i = 1; i <= amount; i++) {
                if(i >= coin)
                    dp[i] += dp[i - coin];
            }
        }
        return dp[amount];
    }
}
```



# 滑动窗口最大值

## 题目描述

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。

**示例 1：**

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**示例 2：**

```
输入：nums = [1], k = 1
输出：[1]
```



## 题解

解法一：暴力求解，在LeetCode中超时，但是面试AC了，抽象测例这一块

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        int len = n - k + 1;
        int[] result = new int[len];
        for(int i = 0; i < n - k + 1; i++) {
            int j = i + k - 1;
            int max = nums[i];
            for(int t = i; t <= j ;t++) {
                max = Math.max(max, nums[t]);
            }
            result[i] = max;
        }
        return result;
    }
}
```

解法二：通过单调队列求解

设立一个双端队列用于存储元素索引，遍历数组，其中 j 代表当前加入的元素对应的最前端索引位置

两次循环目的分别是删除不在窗口中的索引，和从后向前删除小于当前元素的对应索引，之后将当前元素放入队列末尾，这样就可以维护一个单调递减的队列，且其中放置的元素索引都是在滑动窗口中的元素索引，当满足窗口左边界大于等于0的时候就可以从队列左侧取出最大的元素索引，将元素放入结果集合中

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        Deque<Integer> deque = new LinkedList<>();
        int n = nums.length;
        int[] result = new int[n - k + 1];
        for(int i = 0; i < n; i++) {
            int j = i - k + 1;
            while(!deque.isEmpty() && deque.peekFirst() < j) {
                deque.pollFirst();
            }
            while(!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) {
                deque.pollLast();
            }
            deque.offerLast(i);
            if(j >= 0) {
                result[j] = nums[deque.peekFirst()];
            }
        }
        return result;
    }
}
```

