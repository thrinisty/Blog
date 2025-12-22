---
title: LeetCode刷题（存在重复元素Ⅲ、我的日程安排表）
published: 2025-12-16
updated: 2025-12-16
description: '存在重复元素Ⅲ、我的日程安排表'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 存在重复元素Ⅲ表

## 题目描述

给你一个整数数组 `nums` 和两个整数 `k` 和 `t` 。请你判断是否存在 **两个不同下标** `i` 和 `j`，使得 `abs(nums[i] - nums[j]) <= t` ，同时又满足 `abs(i - j) <= k` 。

如果存在则返回 `true`，不存在返回 `false`。

**示例 1：**

```
输入：nums = [1,2,3,1], k = 3, t = 0
输出：true
```

**示例 2：**

```
输入：nums = [1,0,1,1], k = 1, t = 2
输出：true
```

**示例 3：**

```
输入：nums = [1,5,9,1,5,9], k = 2, t = 3
输出：false
```



## 题解

使用TreeSet集合，这个集合由红黑树构成，可以保证内部元素按照顺序排列，每一次加入元素的时候查询上一个、下一个元素的值并判断值相差小于等于t，如果满足则返回true（前后两个元素不满足则其余的也不满足）

加入当前元素，并移除 i - k 对应的元素，因为下一个i + 1的位置放入元素对于这个i - k的元素是没有办法满足i - j <= k 的

```java
class Solution {
    public boolean containsNearbyAlmostDuplicate(int[] nums, int k, int t) {
        TreeSet<Integer> set = new TreeSet<>();
        for(int i = 0; i < nums.length; i++) {
            Integer num = nums[i];
            Integer left = set.floor(num);
            Integer right = set.ceiling(num);
            if(left != null && (long)num - left <= t) {
                return true;
            }
            if(right != null && (long)right - num <= t) {
                return true;
            }
            set.add(num);
            if(i >= k) {
                set.remove(nums[i - k]);
            }
        }
        return false;
    }
}
```



# 我的日程安排表

## 题目描述

请实现一个 `MyCalendar` 类来存放你的日程安排。如果要添加的时间内没有其他安排，则可以存储这个新的日程安排。

`MyCalendar` 有一个 `book(int start, int end)`方法。它意味着在 start 到 end 时间内增加一个日程安排，注意，这里的时间是半开区间，即 `[start, end)`, 实数 `x` 的范围为，  `start <= x < end`。

当两个日程安排有一些时间上的交叉时（例如两个日程安排都在同一时间内），就会产生重复预订。

每次调用 `MyCalendar.book`方法时，如果可以将日程安排成功添加到日历中而不会导致重复预订，返回 `true`。否则，返回 `false` 并且不要将该日程安排添加到日历中。

请按照以下步骤调用 `MyCalendar` 类: `MyCalendar cal = new MyCalendar();` `MyCalendar.book(start, end)`

**示例 1：**

```
输入:
["MyCalendar","book","book","book"]
[[],[10,20],[15,25],[20,30]]
输出: [null,true,false,true]
解释: 
MyCalendar myCalendar = new MyCalendar();
MyCalendar.book(10, 20); // returns true 
MyCalendar.book(15, 25); // returns false ，第二个日程安排不能添加到日历中，因为时间 15 已经被第一个日程安排预定了
MyCalendar.book(20, 30); // returns true ，第三个日程安排可以添加到日历中，因为第一个日程安排并不包含时间 20 
```



## 题解

解法一：运用List集合存储，每次放入的时候遍历集合看是否有冲突

```java
class MyCalendar {
    List<int[]> booked;
    public MyCalendar() {
        booked = new ArrayList<>();
    }
    
    public boolean book(int start, int end) {
        for(int[] arr : booked) {
            int left = arr[0];
            int right = arr[1];
            if(end > left && start < right) {
                return false;
            }
        }
        booked.add(new int[]{start, end});
        return true;
    }
}
```

解法二：运用TreeSet设置比较规则，因为集合中的元素都满足不重叠，只需要设置比较起始从小到大即可，取出前一个和后一个位置的元素，元素存在时判断是否冲突即可

```java
class MyCalendar {
    TreeSet<int[]> booked;

    public MyCalendar() {
        booked = new TreeSet<>((a, b) -> a[0] - b[0]);
    }
    
    public boolean book(int start, int end) {
        int[] current = new int[]{start, end};
        if(!booked.isEmpty()) {
            int[] prev = booked.floor(current);
            int[] next = booked.ceiling(current);
            if(prev != null && start < prev[1]) {
                return false;
            }
            if(next != null && end > next[0]) {
                return false;
            }
        }
        booked.add(current);
        return true;
    }
}
```

