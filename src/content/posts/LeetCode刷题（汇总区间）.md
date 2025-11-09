---
title: LeetCode刷题（汇总区间）
published: 2025-11-04
updated: 2025-11-04
description: '汇总区间'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 汇总区间

## 题目描述

给定一个  **无重复元素** 的 **有序** 整数数组 `nums` 。

区间 `[a,b]` 是从 `a` 到 `b`（包含）的所有整数的集合。

返回 ***恰好覆盖数组中所有数字** 的 **最小有序** 区间范围列表* 。也就是说，`nums` 的每个元素都恰好被某个区间范围所覆盖，并且不存在属于某个区间但不属于 `nums` 的数字 `x` 。

列表中的每个区间范围 `[a,b]` 应该按如下格式输出：

- `"a->b"` ，如果 `a != b`
- `"a"` ，如果 `a == b`

**示例 1：**

```
输入：nums = [0,1,2,4,5,7]
输出：["0->2","4->5","7"]
解释：区间范围是：
[0,2] --> "0->2"
[4,5] --> "4->5"
[7,7] --> "7"
```



## 题解

通过双指针求解，判断当前元素后续元素值是否是当前元素+1，否则后续元素是另一个区间，将头部和尾部之间比较，如果相同代表只有一个元素，如果不同则区间就是这两个元素，将其加入结果集合中即可

```java
class Solution {
    public List<String> summaryRanges(int[] nums) {
        List<String> result = new ArrayList<>();
        int i = 0;
        for(int j = 0; j < nums.length; j++) {
            if(j + 1 == nums.length || nums[j] + 1 != nums[j + 1]) {
                StringBuilder target = new StringBuilder();
                target.append(nums[i]);
                if(i != j) {
                    target.append("->").append(nums[j]);
                }
                result.add(target.toString());
                i = j + 1;
            }
        }
        return result;
    }
}
```

