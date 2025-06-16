---
title: LeetCode刷题（有效的括号）
published: 2025-06-16
updated: 2025-06-16
description: '有效的括号'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 有效的括号

一样的今天主要还是复习软件工程，做两道简单的题目

## 题目描述

给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。
3. 每个右括号都有一个对应的相同类型的左括号。

**示例 1：**

**输入：**s = "()"

**输出：**true

**示例 2：**

**输入：**s = "()[]{}"

**输出：**true

**示例 3：**

**输入：**s = "(]"

**输出：**false

**示例 4：**

**输入：**s = "([])"

**输出：**true



## 题解

设立一个栈结构，依次遍历字符串的字符，为左括号入栈，如果不是左括号，先判断栈是否为空，为空返回false，之后弹栈用弹出的元素比较是否和入站的元素匹配，不匹配则返回false，否则继续遍历

当字符串遍历完成的时候，如果合法，栈应该空的

```java
class Solution {
    public boolean isValid(String s) {
        if(s.length() % 2 != 0) {
            return false;
        }
        Map<Character, Character> map = new HashMap<>();
        map.put(')', '(');
        map.put(']', '[');
        map.put('}', '{');
        Stack<Character> stack = new Stack<>();
        for(char c : s.toCharArray()) {
            if(c == '(' || c == '[' || c == '{') {
                stack.push(c);
            } else if(stack.isEmpty() || stack.pop() != map.get(c)) {
                return false;
            } else {
                continue;
            }
        }
        return stack.isEmpty();
    }
}
```



# 爬楼梯

## 题目描述

假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。

每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？

**示例 1：**

```
输入：n = 2
输出：2
解释：有两种方法可以爬到楼顶。
1. 1 阶 + 1 阶
2. 2 阶
```

**示例 2：**

```
输入：n = 3
输出：3
解释：有三种方法可以爬到楼顶。
1. 1 阶 + 1 阶 + 1 阶
2. 1 阶 + 2 阶
3. 2 阶 + 1 阶
```



## 题解

本质上类似斐波那契数列的求解，我们运用动态规划做题

设跳上 n 级台阶有 f(n) 种跳法。在所有跳法中，青蛙的最后一步只有两种情况： 跳上 1 级或 2 级台阶。

当为 1 级台阶： 剩 n−1 个台阶，此情况共有 f(n−1) 种跳法。
当为 2 级台阶： 剩 n−2 个台阶，此情况共有 f(n−2) 种跳法。
即 f(n) 为以上两种情况之和，即 f(n)=f(n−1)+f(n−2) ，以上递推性质为斐波那契数列。因此，本题可转化为 求斐波那契数列的第 n 项，区别仅在于初始值不同：

青蛙跳台阶问题： f(0)=1 , f(1)=1 , f(2)=2 。
斐波那契数列问题： f(0)=0 , f(1)=1 , f(2)=1 。

```java
class Solution {
    public int climbStairs(int n) {
        int[] result = new int[n + 1];
        result[0] = 1;
        result[1] = 1;
        for(int i = 2; i <= n; i++) {
            result[i] = result[i - 1] + result[i - 2];
        }
        return result[n];
    }
}
```

因为只需要存储最后的结果，我们也可以用迭代法求解

```java
class Solution {
    public int climbStairs(int n) {
        int a = 1, b = 1, sum;
        for(int i = 0; i < n - 1; i++) {
            sum = a + b;
            a = b;
            b = sum;
        }
        return b;
    }
}
```

