---
title: LeetCode刷题（查找和最小的K对数字、基本计算器、缺失的第一个正数）
published: 2025-11-28
updated: 2025-11-28
description: '查找和最小的K对数字、基本计算器、缺失的第一个正数'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 查找和最小的K对数字

## 题目描述

给定两个以 **非递减顺序排列** 的整数数组 `nums1` 和 `nums2` , 以及一个整数 `k` 。

定义一对值 `(u,v)`，其中第一个元素来自 `nums1`，第二个元素来自 `nums2` 。

请找到和最小的 `k` 个数对 `(u1,v1)`, ` (u2,v2)` ...  `(uk,vk)` 。

**示例 1:**

```
输入: nums1 = [1,7,11], nums2 = [2,4,6], k = 3
输出: [1,2],[1,4],[1,6]
解释: 返回序列中的前 3 对数：
     [1,2],[1,4],[1,6],[7,2],[7,4],[11,2],[7,6],[11,4],[11,6]
```

**示例 2:**

```
输入: nums1 = [1,1,2], nums2 = [1,2,3], k = 2
输出: [1,1],[1,1]
解释: 返回序列中的前 2 对数：
     [1,1],[1,1],[1,2],[2,1],[1,2],[2,2],[1,3],[1,3],[2,3]
```



## 题解

运用最小堆进行求解，由于两个数组都是单调递增的，对于由两个数组分别作为下标得到和的二维数组满足从左到右依次递增以及从上至下依次递增，将第一列的元素放入最小堆中，依次从最小堆中取出最小的和以及对应的横纵坐标，将其右侧的元素和以及横纵坐标放入最小堆中，取够k个数即可

```java
class Solution {
    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        for(int i = 0; i < Math.min(nums1.length, k); i++) {
            heap.offer(new int[]{nums1[i] + nums2[0], i, 0});
        }
        
        List<List<Integer>> result = new ArrayList<>();
        for(int t = 0; t < k; t++) {
            int[] min = heap.poll();
            int i = min[1];
            int j = min[2];
            result.add(Arrays.asList(nums1[i], nums2[j]));
            if(j < nums2.length - 1) {
                heap.offer(new int[]{nums1[i] + nums2[j + 1], i, j + 1});
            }
        }
        return result;
    }
}
```



# 基本计算器

## 题目描述

给你一个字符串表达式 `s` ，请你实现一个基本计算器来计算并返回它的值。

注意:不允许使用任何将字符串作为数学表达式计算的内置函数，比如 `eval()` 。

**示例 1：**

```
输入：s = "1 + 1"
输出：2
```

**示例 2：**

```
输入：s = " 2-1 + 2 "
输出：3
```

**示例 3：**

```
输入：s = "(1+(4+5+2)-3)+(6+8)"
输出：23
```



## 题解

本质上对于每个数而言无外乎取正取负，通过sign作为其符号标识，再额外设立一个栈标识对应括号范围内的符号取值

```
当前字符为+的时候就直接从栈中取出符号（正或负）
如果是-则peek出来的符号取反
如果是(则需要将当前的符号放入栈中，用作当前括号范围内数据取符号的基准
如果是)则当前的括号完毕，将其弹出，之后的数字基准为之前括号的符号
```

```java
class Solution {
    public int calculate(String s) {
        Stack<Integer> ops = new Stack<>();
        ops.push(1);
        int sign = 1;
        
        int result = 0;
        int i = 0;
        int n = s.length();
        while(i < n) {
            char c = s.charAt(i);
            if(c == ' ') {
                i++;
            } else if(c == '+') {
                sign = ops.peek();
                i++;
            } else if(c == '-') {
                sign = -ops.peek();
                i++;
            } else if(c == '(') {
                ops.push(sign);
                i++;
            } else if(c == ')') {
                ops.pop();
                i++;
            } else {
                long num = 0;
                while(i < n && s.charAt(i) >= '0' && s.charAt(i) <= '9') {
                    num = num * 10 + s.charAt(i) - '0';
                    i++;
                }
                result += sign * num;
            }
        }
        return result;
    }
}
```



# 缺失的第一个正数

## 题目描述

给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。

请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。

**示例 1：**

```
输入：nums = [1,2,0]
输出：3
解释：范围 [1,2] 中的数字都在数组中。
```

**示例 2：**

```
输入：nums = [3,4,-1,1]
输出：2
解释：1 在数组中，但 2 没有。
```

**示例 3：**

```
输入：nums = [7,8,9,11,12]
输出：1
解释：最小的正数 1 没有出现。
```



## 题解

我们的所有数字的预期是1-n，即从下标0开始到n-1都对应着下标+1的数

遍历数组，对于遍历到的数字满足在数组预期的范围内且不满足其位置在自己应该在的位置上的时候，需要交换将其放在应该在的位置上，对于交换过来的数字也应该重新判断（while），等全部交换完毕的时候如果数字是预期放入的数字，那么它们就在自己的位置上

这样只需要从头开始，找到第一个不符合预期的数字位置，返回位置 + 1，就是缺少的数字

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int n = nums.length;
        for(int i = 0; i < n; i++) {
            while(nums[i] >= 1 && nums[i] <= n && nums[i] - 1 != i) {
                int j = nums[i] - 1;
                int temp = nums[i];
                nums[i] = nums[j];
                nums[j] = temp;
            }
        }
        for(int i = 0; i < n; i++) {
            if(nums[i] != i + 1) {
                return i + 1;
            }
        }
        return n + 1;
    }
}
```

但是这种方式没有办法处理有重复元素出现的情况，会出现死循环，这个时候需要更改判断条件，在相同的时候直接break即可

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int n = nums.length;
        for(int i = 0; i < n; i++) {
            while(nums[i] >= 1 && nums[i] <= n && nums[i] - 1 != i) {
                int j = nums[i] - 1;
                if(nums[i] == nums[j]) break;
                int temp = nums[i];
                nums[i] = nums[j];
                nums[j] = temp;
            }
        }
        for(int i = 0; i < n; i++) {
            if(nums[i] != i + 1) {
                return i + 1;
            }
        }
        return n + 1;
    }
}
```

