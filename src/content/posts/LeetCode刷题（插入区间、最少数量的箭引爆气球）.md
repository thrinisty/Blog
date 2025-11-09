---
title: LeetCode刷题（插入区间、最少数量的箭引爆气球）
published: 2025-11-05
updated: 2025-11-05
description: '插入区间、最少数量的箭引爆气球'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 插入区间

## 题目描述

给你一个 **无重叠的** *，*按照区间起始端点排序的区间列表 `intervals`，其中 `intervals[i] = [starti, endi]` 表示第 `i` 个区间的开始和结束，并且 `intervals` 按照 `starti` 升序排列。同样给定一个区间 `newInterval = [start, end]` 表示另一个区间的开始和结束。

在 `intervals` 中插入区间 `newInterval`，使得 `intervals` 依然按照 `starti` 升序排列，且区间之间不重叠（如果有必要的话，可以合并区间）。

返回插入之后的 `intervals`。

**示例 1：**

```
输入：intervals = [[1,3],[6,9]], newInterval = [2,5]
输出：[[1,5],[6,9]]
```

**示例 2：**

```
输入：intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
输出：[[1,2],[3,10],[12,16]]
解释：这是因为新的区间 [4,8] 与 [3,5],[6,7],[8,10] 重叠。
```



## 题解

还是用之前的方式，按照顺序插入，并合并

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        if(intervals == null || intervals.length == 0) {
            return new int[][]{newInterval};
        }
        List<int[]> result = new ArrayList<>();
        int index = -1;
        for(int i = 0; i < intervals.length; i++) {
            int[] range = intervals[i];
            if(range[0] >= newInterval[0]) {
                index = i;
                break;
            }
        }
        if(newInterval[0] > intervals[intervals.length - 1][0]) {
            index = -2;
        }

        if(index == -1) {
            result.add(newInterval);
        }

        for(int i = 0; i < intervals.length; i++) {
            if(i == index) {
                if(result.isEmpty() || newInterval[0] > result.get(result.size() - 1)[1]) {
                    result.add(newInterval);
                } else {
                    int[] target = result.get(result.size() - 1);
                    target[1] = Math.max(target[1], newInterval[1]);
                }
            }

            if(result.isEmpty() || intervals[i][0] > result.get(result.size() - 1)[1]) {
                result.add(intervals[i]);
            } else {
                int[] target = result.get(result.size() - 1);
                target[1] = Math.max(target[1], intervals[i][1]);
            }
        }
        if(index == -2) {
            if(result.isEmpty() || newInterval[0] > result.get(result.size() - 1)[1]) {
                    result.add(newInterval);
                } else {
                    int[] target = result.get(result.size() - 1);
                    target[1] = Math.max(target[1], newInterval[1]);
                }
        }
        int[][] ans = new int[result.size()][2];
        for(int i = 0; i < result.size(); i++) {
            ans[i] = result.get(i);
        }
        return ans;
    }
}
```



通过较为简单的方式实现

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]> result = new ArrayList<>();
        int i = 0;
        int n = intervals.length;
        
        // 1. 添加所有结束时间小于新区间开始时间的区间
        while (i < n && intervals[i][1] < newInterval[0]) {
            result.add(intervals[i]);
            i++;
        }
        
        // 2. 合并所有与新区间重叠的区间
        while (i < n && intervals[i][0] <= newInterval[1]) {
            newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
            newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
            i++;
        }
        result.add(newInterval);
        
        // 3. 添加剩余区间
        while (i < n) {
            result.add(intervals[i]);
            i++;
        }
        
        return result.toArray(new int[result.size()][]);
    }
}
```



# 最少数量的箭引爆气球

## 题目描述

有一些球形气球贴在一堵用 XY 平面表示的墙面上。墙面上的气球记录在整数数组 `points` ，其中`points[i] = [xstart, xend]` 表示水平直径在 `xstart` 和 `xend`之间的气球。你不知道气球的确切 y 坐标。

一支弓箭可以沿着 x 轴从不同点 **完全垂直** 地射出。在坐标 `x` 处射出一支箭，若有一个气球的直径的开始和结束坐标为 `xstart`，`xend`， 且满足  `xstart ≤ x ≤ xend`，则该气球会被 **引爆** 。可以射出的弓箭的数量 **没有限制** 。 弓箭一旦被射出之后，可以无限地前进。

给你一个数组 `points` ，*返回引爆所有气球所必须射出的 **最小** 弓箭数* 。

**示例 1：**

```
输入：points = [[10,16],[2,8],[1,6],[7,12]]
输出：2
解释：气球可以用2支箭来爆破:
-在x = 6处射出箭，击破气球[2,8]和[1,6]。
-在x = 11处发射箭，击破气球[10,16]和[7,12]。
```

**示例 2：**

```
输入：points = [[1,2],[3,4],[5,6],[7,8]]
输出：4
解释：每个气球需要射出一支箭，总共需要4支箭。
```

**示例 3：**

```
输入：points = [[1,2],[2,3],[3,4],[4,5]]
输出：2
解释：气球可以用2支箭来爆破:
- 在x = 2处发射箭，击破气球[1,2]和[2,3]。
- 在x = 4处射出箭，击破气球[3,4]和[4,5]。
```



## 题解

先进行按照起始进行从小到大的排序，如果当前区间的左边小于上一个区间的右边界，代表这两个区间重合，可以一并射穿，我们把区间设置为交集供下一个区间判断是否可以合并

```java
class Solution {
    public int findMinArrowShots(int[][] points) {
        if(points.length == 0) {
            return 0;
        }
        Arrays.sort(points, (p1, p2) -> Integer.compare(p1[0], p2[0]));
        int n = points.length;
        for(int i = 1; i < points.length; i++) {
            if(points[i][0] <= points[i - 1][1]) {
                n--;
                points[i][0] = Math.max(points[i][0], points[i - 1][0]);
                points[i][1] = Math.min(points[i][1], points[i - 1][1]);
            }
        }
        return n;
    }
}
```



