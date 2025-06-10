---
title: LeetCode刷题（电话号码的字母组合，打家劫舍）
published: 2025-06-09
updated: 2025-06-09
description: '电话号码的字母组合，打家劫舍'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 电话号码的字母组合

## 题目描述

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按 **任意顺序** 返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![184](../images/184.png)

**示例 1：**

```
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

**示例 2：**

```
输入：digits = ""
输出：[]
```

**示例 3：**

```
输入：digits = "2"
输出：["a","b","c"]
```

**提示：**

- `0 <= digits.length <= 4`
- `digits[i]` 是范围 `['2', '9']` 的一个数字。



## 题解

自己写的，参考了之前的有一道括号组合的方式，用temp数组映射出对应的字符串，挨个取出递归加入之前的字符串，但是执行速度惨淡，好在可以过关

想了想应该是没有用StringBuffer的缘故，重复创建字符串到缓冲池，导致的执行速度和空间存储都很差，先这样吧，之后再找时间解决

```java
class Solution {
    private String[] map = {
        "",//0
        "",//1
        "abc",//2
        "def",//3
        "ghi",//4
        "jkl",//5
        "mno",//6
        "pqrs",//7
        "tuv",//8
        "wxyz"//9
    };
    List<String> result = new ArrayList<>();
    public List<String> letterCombinations(String digits) {
        fun("", digits, 0);
        return result;
    }
    public void fun(String str, String digits, int idx) {
        if(idx == digits.length()) {
            if(str != "") {
                result.add(str);
            }
            return;
        }
        int flag = digits.charAt(idx) - '0';
        String target = map[flag];
        for(int i = 0; i < target.length(); i++) {
            char c = target.charAt(i);
            fun(str + c, digits, idx + 1);
        }
    }
}
```



# 打家劫舍

## 题目描述

小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为 `root` 。

除了 `root` 之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果 **两个直接相连的房子在同一天晚上被打劫** ，房屋将自动报警。

给定二叉树的 `root` 。返回 ***在不触动警报的情况下** ，小偷能够盗取的最高金额* 。

 

**示例 1:**

![185](../images/185.jpg)

```
输入: root = [3,2,3,null,3,null,1]
输出: 7 
解释: 小偷一晚能够盗取的最高金额 3 + 3 + 1 = 7
```

**示例 2:**

![186](../images/186.jpg)

```
输入: root = [3,4,5,1,3,null,1]
输出: 9
解释: 小偷一晚能够盗取的最高金额 4 + 5 = 9
```

## 题解

解法一：递归的判断是否打劫当前人家，返回最大的金钱数

```java
class Solution {
    public int rob(TreeNode root) {
        if(root == null){
            return 0;
        }
        int money = root.val;
        if(root.left != null) {
            money += rob(root.left.left) + rob(root.left.right);
        }
        if(root.right != null) {
            money += rob(root.right.left) + rob(root.right.right);
        }
        return Math.max(money, rob(root.left) + rob(root.right));
    }
}
```

这种方法在递归的时候会产生非常多的重复情况，会出现超时

解法二：使用一个递归函数，返回一个大小为2的结果集，0表示不打劫该人家的最大金钱数，而1表示打劫该人家的最大金钱数

```java
class Solution {
    public int rob(TreeNode root) {
        int[] result = robArray(root);
        return Math.max(result[0], result[1]);
    }

    public int[] robArray(TreeNode root) {
        int[] result = new int[2];
        if(root == null) {
            return result;
        }
        int[] left = robArray(root.left);
        int[] right = robArray(root.right);
        result[0] = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
        result[1] = root.val + left[0] + right[0];
        return result;
    }
}
```

