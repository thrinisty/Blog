---
title: LeetCode刷题（无重复字符的最长字串、寻找两个正序数组中的中位数）
published: 2025-10-07
updated: 2025-10-07
description: '无重复字符的最长字串、寻找两个正序数组中的中位数'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 无重复字符的最长字串

## 题目描述

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长 子串** 的长度。

**示例 1:**

```
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**示例 2:**

```
输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

**示例 3:**

```
输入: s = "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```



## 题解

解法一：我记得原先做过一道用boolean[26]存储是否被存储的题目，我们可以使用Set来构造是否被放入，如果被放入，需要依次将Set中的元素取出直至元素中不含有c为止

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        Set<Character> set = new HashSet<>();
        int result = 0;
        int left = 0;
        int n = s.length();
        for(int right = 0; right < n; right++) {
            char c = s.charAt(right);
            while(set.contains(c)) {
                set.remove(s.charAt(left));
                left++;
            }
            set.add(c);
            result = Math.max(result, right - left + 1);
        }
        return result;
    }
}
```



解法二：

构造一个HashMap其中key存字符数值，value存放对应的下表，如果当前遍历元素中被mapKey包含，则证明加上该字符后重复，需要更新左边界，从map中取出对应的值即为左边界，（字符串集不包含左边界）

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        Map<Character, Integer> map = new HashMap<>();
        int result = 0;
        int n = s.length();
        int i = -1;
        for(int j = 0; j < n; j++) {
            char c = s.charAt(j);
            if(map.containsKey(c)) {
                i = Math.max(i, map.get(c));
            }
            map.put(c, j);
            result = Math.max(result, j - i);
        }
        return result;
    }
}
```

其实思路上大体一致，就是滑动窗口，区别于后者是会存入对应的下表，取得时候不用删除元素至没有重复元素

# 寻找两个正序数组中的中位数

## 题目描述

给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数** 。

算法的时间复杂度应该为 `O(log (m+n))` 。

**示例 1：**

```
输入：nums1 = [1,3], nums2 = [2]
输出：2.00000
解释：合并数组 = [1,2,3] ，中位数 2
```

**示例 2：**

```
输入：nums1 = [1,2], nums2 = [3,4]
输出：2.50000
解释：合并数组 = [1,2,3,4] ，中位数 (2 + 3) / 2 = 2.5
```



## 题解

解法一：组合两个数组，因为顺序是单调的，我们可以利用类似合并二叉树哪一题的方式合成一个单调递增的数组，取left和right/2即可，时间复杂度为O(m + n)，因为合并两个数组需要遍历完整数组

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int m = nums1.length;
        int n = nums2.length;
        int length = m + n;
        int[] array = new int[length];
        int i = 0;
        int j = 0;
        int index = 0;
        while(i < m && j < n) {
            if(nums1[i] < nums2[j]) {
                array[index++] = nums1[i++];
            } else{
                array[index++] = nums2[j++];
            }
        }
        while(i < m) {
            array[index++] = nums1[i++];
        }
        while(j < n) {
            array[index++] = nums2[j++];
        }
        int left = (length - 1) / 2 ;
        int right = length / 2;
        return (double)(array[left] + array[right]) / 2;
    }
}
```



解法二：二分递归，定义一个方法，传入两个数组，以及对应的索引，需要获取到的第k个数，返回第k个数的值

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int len  = nums1.length + nums2.length;
        if(len % 2 == 1){
            return findK(nums1, nums2, 0, 0, len / 2 + 1);
        } else {
            return (findK(nums1, nums2, 0, 0, len / 2 + 1) + findK(nums1, nums2, 0, 0, len / 2)) / 2.0;
        }
    }

    public int findK(int[] nums1, int[] nums2, int i, int j, int k) {
        int n1 = nums1.length;
        int n2 = nums2.length;
        if(n1 - i > n2 - j) {
            return findK(nums2, nums1, j, i, k);
        }
        if(n1 == i) {
            return nums2[j + k - 1];
        }
        if(k == 1) {
            return Math.min(nums1[i], nums2[j]);
        }
        int mid1 = Math.min(i + k / 2, n1);
        int mid2 = j + k / 2;
        if(nums1[mid1 - 1] > nums2[mid2 - 1]) {
            return findK(nums1, nums2, i, mid2, k - (mid2 - j));
        } else {
            return findK(nums1, nums2, mid1, j, k - (mid1 - i));
        }
    }
}
```

