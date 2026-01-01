---
title: LeetCode刷题（复原IP地址、使用最小花费爬楼梯）
published: 2025-12-28
updated: 2025-12-28
description: '复原IP地址、使用最小花费爬楼梯'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 复原IP地址

## 题目描述

给定一个只包含数字的字符串 `s` ，用以表示一个 IP 地址，返回所有可能从 `s` 获得的 **有效 IP 地址** 。你可以按任何顺序返回答案。

**有效 IP 地址** 正好由四个整数（每个整数位于 0 到 255 之间组成，且不能含有前导 `0`），整数之间用 `'.'` 分隔。

例如："0.1.2.201" 和 "192.168.1.1" 是 **有效** IP 地址，但是 "0.011.255.245"、"192.168.1.312" 和 "192.168@1.1" 是 **无效** IP 地址。

**示例 1：**

```
输入：s = "25525511135"
输出：["255.255.11.135","255.255.111.35"]
```

**示例 2：**

```
输入：s = "0000"
输出：["0.0.0.0"]
```

**示例 3：**

```
输入：s = "1111"
输出：["1.1.1.1"]
```

**示例 4：**

```
输入：s = "010010"
输出：["0.10.0.10","0.100.1.0"]
```

**示例 5：**

```
输入：s = "10203040"
输出：["10.20.30.40","102.0.30.40","10.203.0.40"]
```



## 题解

运用回溯法求解，当list结果中有4个元素，且index结束的时候加入result，如果超出了4个元素单结果也没取完则不加入result并return

每次回溯取出至多三个字符串并依次判断是否合法，合法则加入list并进行下一次回溯

```java
class Solution {
    List<String> result = new ArrayList<>();
    List<String> list = new ArrayList<>();
    public List<String> restoreIpAddresses(String s) {
        dfs(s, 0);
        return result;
    }

    public void dfs(String s, int index) {
        if(list.size() == 4) {
            if(index == s.length()) {
                result.add(restoreIp(list));
            }
            return;
        } 
        int most = Math.min(s.length(), index + 3);
        for(int i = index; i < most; i++) {
            String str = s.substring(index, i + 1);
            int num = Integer.parseInt(str);
            if(str.charAt(0) == '0' && str.length() > 1 || num < 0 || num > 255) {
                break;
            }
            list.add(str);
            dfs(s, i + 1);
            list.remove(list.size() - 1);
        }
    }

    public String restoreIp(List<String> list) {
        StringBuilder str = new StringBuilder();
        int n = list.size();
        for(int i = 0; i < n - 1; i++) {
            str.append(list.get(i));
            str.append(".");
        }
        str.append(list.get(n - 1));
        return str.toString();
    }
}
```



# 使用最小花费爬楼梯

## 题目描述

数组的每个下标作为一个阶梯，第 `i` 个阶梯对应着一个非负数的体力花费值 `cost[i]`（下标从 `0` 开始）。

每当爬上一个阶梯都要花费对应的体力值，一旦支付了相应的体力值，就可以选择向上爬一个阶梯或者爬两个阶梯。

请找出达到楼层顶部的最低花费。在开始时，你可以选择从下标为 0 或 1 的元素作为初始阶梯。

**示例 1：**

```
输入：cost = [10, 15, 20]
输出：15
解释：最低花费是从 cost[1] 开始，然后走两步即可到阶梯顶，一共花费 15 。
```

 **示例 2：**

```
输入：cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1]
输出：6
解释：最低花费方式是从 cost[0] 开始，逐个经过那些 1 ，跳过 cost[3] ，一共花费 6 。
```



## 题解

动态规划求解，最后取最后一层和倒数第二层的最小dp值即可

```java
class Solution {
    public int minCostClimbingStairs(int[] cost) {
        int n = cost.length;
        int[] dp = new int[n + 1];
        dp[0] = cost[0];
        dp[1] = cost[1];
        for(int i = 2; i < n; i++) {
            dp[i] = Math.min(dp[i - 1], dp[i - 2]) + cost[i];
        }
        return Math.min(dp[n - 1], dp[n - 2]);
    }
}
```

