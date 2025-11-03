---
title: LeetCode刷题（快乐数、存在重复元素Ⅱ）
published: 2025-11-03
updated: 2025-11-03
description: '快乐数、存在重复元素Ⅱ'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 快乐数

## 题目描述

编写一个算法来判断一个数 `n` 是不是快乐数。

**「快乐数」** 定义为：

- 对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。
- 然后重复这个过程直到这个数变为 1，也可能是 **无限循环** 但始终变不到 1。
- 如果这个过程 **结果为** 1，那么这个数就是快乐数。

如果 `n` 是 *快乐数* 就返回 `true` ；不是，则返回 `false` 。

**示例 1：**

```
输入：n = 19
输出：true
解释：
12 + 92 = 82
82 + 22 = 68
62 + 82 = 100
12 + 02 + 02 = 1
```

**示例 2：**

```
输入：n = 2
输出：false
```



## 题解

解法一：通过快慢指针进行求解

```java
class Solution {
    public boolean isHappy(int n) {
        int slow = n, fast = getNext(n);
        while(true) {
            slow = getNext(slow);
            fast = getNext(getNext(fast));
            if(fast == slow) {
                break;
            }
        }
        return fast == 1;
    }

    public int getNext(int n) {
        int sum = 0;
        while(n != 0) {
            int dev = n % 10;
            n /= 10;
            sum += dev * dev;
        }
        return sum;
    }
}
```



解法二：通过Set进行判断是否有重复元素，有则停止，判断当前数是否为1

```java
class Solution {
    public boolean isHappy(int n) {
        Set<Integer> set = new HashSet<>();
        while(!set.contains(n)) {
            set.add(n);
            n = getNext(n);
        }
        return n == 1;
    }

    public int getNext(int n) {
        int sum = 0;
        while(n != 0) {
            int dev = n % 10;
            n /= 10;
            sum += dev * dev;
        }
        return sum;
    }
}
```



# 存在重复元素Ⅱ

## 题目描述

给你一个整数数组 `nums` 和一个整数 `k` ，判断数组中是否存在两个 **不同的索引** `i` 和 `j` ，满足 `nums[i] == nums[j]` 且 `abs(i - j) <= k` 。如果存在，返回 `true` ；否则，返回 `false` 

**示例 1：**

```
输入：nums = [1,2,3,1], k = 3
输出：true
```

**示例 2：**

```
输入：nums = [1,0,1,1], k = 1
输出：true
```

**示例 3：**

```
输入：nums = [1,2,3,1,2,3], k = 2
输出：false
```



## 题解

通过哈希表进行求解，思路和两数之和大差不差

```java
class Solution {
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < nums.length; i++) {
            if(map.containsKey(nums[i])) {
                if(Math.abs(map.get(nums[i]) - i) <= k) {
                    return true;
                }
            }
            map.put(nums[i], i);
        }
        return false;
    }
}
```

