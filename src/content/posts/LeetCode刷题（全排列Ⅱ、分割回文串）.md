---
title: LeetCode刷题（全排列Ⅱ、分割回文串）
published: 2025-12-27
updated: 2025-12-27
description: '全排列Ⅱ、分割回文串'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 全排列Ⅱ

## 题目描述

给定一个可包含重复数字的整数集合 `nums` ，**按任意顺序** 返回它所有不重复的全排列。

**示例 1：**

```
输入：nums = [1,1,2]
输出：
[[1,1,2],
 [1,2,1],
 [2,1,1]]
```

**示例 2：**

```
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```



## 题解

先对数组排序，然后设置一个辅助的visit数组记录用过的元素，每次调用dfs函数都尝试将所有的元素放入list中，如果其中有元素被用过了则排除当前情况，另外如果当前元素和前一个元素都没有被用过，且他们数值相等则跳过

放入list进行放置下一个数字，放完后删除，达到回溯

```java
class Solution {
    List<List<Integer>> result = new LinkedList<>();
    List<Integer> list = new LinkedList<>();
    boolean[] visit;

    public List<List<Integer>> permuteUnique(int[] nums) {
        Arrays.sort(nums);
        visit = new boolean[nums.length];
        dfs(nums);
        return result;
    }

    public void dfs(int[] nums) {
        if(list.size() == nums.length) {
            result.add(new LinkedList<>(list));
            return;
        }
        for(int i = 0; i < nums.length; i++) {
            if(visit[i]) {
                continue;
            }
            if(i > 0 && !visit[i] && !visit[i - 1] && nums[i] == nums[i - 1]) {
                continue;
            }
            list.add(nums[i]);
            visit[i] = true;
            dfs(nums);
            visit[i] = false;
            list.remove(list.size() - 1);
        }
    }
}
```

# 分割回文串

## 题目描述

给定一个字符串 `s` ，请将 `s` 分割成一些子串，使每个子串都是 **回文串** ，返回 s 所有可能的分割方案。

**回文串** 是正着读和反着读都一样的字符串。

**示例 1：**

```
输入：s = "google"
输出：[["g","o","o","g","l","e"],["g","oo","g","l","e"],["goog","l","e"]]
```

**示例 2：**

```
输入：s = "aab"
输出：[["a","a","b"],["aa","b"]]
```

**示例 3：**

```
输入：s = "a"
输出：[["a"]]
```



## 题解

先运用动态规划求出dp，再使用回溯进行依次判断取出所有的可能

```java
class Solution {
    List<List<String>> result = new LinkedList<>();
    List<String> list = new LinkedList<>();
    boolean[][] dp;

    public List<List<String>> partition(String s) {
        int n = s.length();
        dp = new boolean[n][n];
        for(int len = 1; len <= n; len++) {
            for(int left = 0; left <= n - len; left++) {
                int right = left + len - 1;
                if((s.charAt(left) == s.charAt(right)) && (right - left < 2 || dp[left + 1][right - 1])) {
                    dp[left][right] = true;
                }
            }
        }
        dfs(s, 0);
        return result;
    }

    public void dfs(String s, int index) {
        if(s.length() == index) {
            result.add(new LinkedList<>(list));
            return;
        }
        for(int i = index; i < s.length(); i++) {
            if(dp[index][i]) {
                list.add(s.substring(index, i + 1));
                dfs(s, i + 1);
                list.remove(list.size() - 1);
            }
        }
    }
}
```

