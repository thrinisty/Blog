---
title: LeetCode刷题（寻找数组的中心下标、连续数组）
published: 2025-12-03
updated: 2025-12-03
description: '寻找数组的中心下标、连续数组'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 寻找数组的中心下标

## 题目描述

给你一个整数数组 `nums` ，请计算数组的 **中心下标** 。

数组 **中心下标** 是数组的一个下标，其左侧所有元素相加的和等于右侧所有元素相加的和。

如果中心下标位于数组最左端，那么左侧数之和视为 `0` ，因为在下标的左侧不存在元素。这一点对于中心下标位于数组最右端同样适用。

如果数组有多个中心下标，应该返回 **最靠近左边** 的那一个。如果数组不存在中心下标，返回 `-1` 。

**示例 1：**

```
输入：nums = [1,7,3,6,5,6]
输出：3
解释：
中心下标是 3 。
左侧数之和 sum = nums[0] + nums[1] + nums[2] = 1 + 7 + 3 = 11 ，
右侧数之和 sum = nums[4] + nums[5] = 5 + 6 = 11 ，二者相等。
```

**示例 2：**

```
输入：nums = [1, 2, 3]
输出：-1
解释：
数组中不存在满足此条件的中心下标。
```

**示例 3：**

```
输入：nums = [2, 1, -1]
输出：0
解释：
中心下标是 0 。
左侧数之和 sum = 0 ，（下标 0 左侧不存在元素），
右侧数之和 sum = nums[1] + nums[2] = 1 + -1 = 0 。
```



## 题解

选取每个元素分别计算左右两侧的元素和，如果按照暴力求解要重复计算时间复杂度达到了O(n)

对此可以进行优化，一开始左侧的和为0，右侧的和为整个数组的元素和，每遍历一个元素，左边的和加上这个元素左侧的数，右边的和减去当前的元素，遍历过程中如果有相等的情况则返回i，否则遍历完成后证明没有满足可能，返回-1

```java
class Solution {
    public int pivotIndex(int[] nums) {
        int n = nums.length;
        int leftSum = 0;
        int rightSum = 0;
        for(int num : nums) {
            rightSum += num;
        }
        for(int i = 0; i < n; i++) {
            leftSum += (i > 0 ? nums[i - 1] : 0);
            rightSum -= nums[i];
            if(leftSum == rightSum) {
                return i;
            }
        }
        return -1;
    }
}
```



# 连续数组

## 题目描述

给定一个二进制数组 `nums` , 找到含有相同数量的 `0` 和 `1` 的最长连续子数组，并返回该子数组的长度。

**示例 1：**

```
输入: nums = [0,1]
输出: 2
解释: [0, 1] 是具有相同数量 0 和 1 的最长连续子数组。
```

**示例 2：**

```
输入: nums = [0,1,0]
输出: 2
解释: [0, 1] (或 [1, 0]) 是具有相同数量 0 和 1 的最长连续子数组。
```

 

**提示：**

- `1 <= nums.length <= 105`
- `nums[i]` 不是 `0` 就是 `1`

 

## 题解

运用前缀和于哈希表进行辅助，当num为1的时候加一，不为1的时候减一，代表到了某一个位置向前所有（相加）的总和，并运用HashMap记录第一个该总和的位置，当后续出现相同的count代表，两个位置中间的数其中的0/1是相同的，用数量迭代result结果

```java
class Solution {
    public int findMaxLength(int[] nums) {
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, -1);
        int result = 0;
        int count = 0;
        for(int i = 0; i < nums.length; i++) {
            int num = nums[i];
            if(num == 1) {
                count++;
            } else {
                count--;
            }
            if(map.containsKey(count)) {
                int preIndex = map.get(count);
                result = Math.max(result, i - preIndex);
            } else {
                map.put(count, i);
            }
        }
        return result;
    }
}
```

