---
title: LeetCode刷题（最小时间差、行星碰撞）
published: 2025-12-10
updated: 2025-12-10
description: '最小时间差、行星碰撞'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 最小时间差

## 题目描述

给定一个 24 小时制（小时:分钟 **"HH:MM"**）的时间列表，找出列表中任意两个时间的最小时间差并以分钟数表示。

**示例 1：**

```
输入：timePoints = ["23:59","00:00"]
输出：1
```

**示例 2：**

```
输入：timePoints = ["00:00","23:59","00:00"]
输出：0
```



## 题解

将每个时间转换为秒，并进行排序，另外的对于最小的时间加上一天的秒数放在list末尾（有可能最大的数和第二天最小的数差值是结果中最小的），最后依次比较迭代result即可

```java
class Solution {
    public int findMinDifference(List<String> timePoints) {
        if(timePoints.size() > 24 * 60) {
            return 0;
        }
        List<Integer> mins = new ArrayList<>();
        for(String time : timePoints) {
            String[] arr = time.split(":");
            String minute = arr[0];
            String second = arr[1];
            mins.add(Integer.parseInt(minute) * 60 + Integer.parseInt(second));
        }
        Collections.sort(mins);
        mins.add(mins.get(0) + 24 * 60);
        int result = 24 * 60;
        for(int i = 0; i < mins.size() - 1; i++) {
            result = Math.min(result, mins.get(i + 1) - mins.get(i));
        }
        return result;
    }
}
```



# 行星碰撞

## 题目描述

给定一个整数数组 `asteroids`，表示在同一行的小行星。

对于数组中的每一个元素，其绝对值表示小行星的大小，正负表示小行星的移动方向（正表示向右移动，负表示向左移动）。每一颗小行星以相同的速度移动。

找出碰撞后剩下的所有小行星。碰撞规则：两个行星相互碰撞，较小的行星会爆炸。如果两颗行星大小相同，则两颗行星都会爆炸。两颗移动方向相同的行星，永远不会发生碰撞。

**示例 1：**

```
输入：asteroids = [5,10,-5]
输出：[5,10]
解释：10 和 -5 碰撞后只剩下 10 。 5 和 10 永远不会发生碰撞。
```

**示例 2：**

```
输入：asteroids = [8,-8]
输出：[]
解释：8 和 -8 碰撞后，两者都发生爆炸。
```

**示例 3：**

```
输入：asteroids = [10,2,-5]
输出：[10]
解释：2 和 -5 发生碰撞后剩下 -5 。10 和 -5 发生碰撞后剩下 10 。
```

**示例 4：**

```
输入：asteroids = [-2,-1,1,2]
输出：[-2,-1,1,2]
解释：-2 和 -1 向左移动，而 1 和 2 向右移动。 由于移动方向相同的行星不会发生碰撞，所以最终没有行星发生碰撞。 
```



## 题解

运用栈进行求解，在栈中依次放入星球，并维护稳定状态

循环的向前判断当前的星求是否会发送碰撞，如果发送碰撞，则分为三种可能讨论，1.前星球质量大，放入前星球 2.新放入的星球大，放入后星球 3.两个星球一样大，同时销毁

在这里循环的弹栈，同时更新放入的星球，直到不发生冲突，最后放入，而0是碰撞同时销毁的情况不进行放入

```java
class Solution {
    public int[] asteroidCollision(int[] asteroids) {
        Stack<Integer> stack = new Stack<>();
        for(int asteroid : asteroids) {
            while(!stack.isEmpty() && stack.peek() > 0 && asteroid < 0) {
                int left = stack.pop();
                int right = Math.abs(asteroid);
                if(left > right) {
                    asteroid = left;
                } else if(left < right) {
                    continue;
                } else {
                    asteroid = 0;
                }
            }
            if(asteroid != 0) {
                stack.push(asteroid);
            }
        }
        int n = stack.size();
        int[] result = new int[n];
        for(int i = n - 1; i >= 0; i--) {
            result[i] = stack.pop();
        }
        return result;
    }
}
```

