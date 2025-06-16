---
title: LeetCode刷题（环形链表）
published: 2025-06-15
updated: 2025-06-15
description: '环形链表'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

今天要复习软件工程，做两道简单一点的题目

# 环形链表

## 题目描述

**示例 1：**

![175](../images/175.png)

```
输入：head = [3,2,0,-4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。
```

**示例 2：**

![176](../images/176.png)

```
输入：head = [1,2], pos = 0
输出：返回索引为 0 的链表节点
解释：链表中有一个环，其尾部连接到第一个节点。
```

**示例 3：**

![178](../images/178.png)

```
输入：head = [1], pos = -1
输出：返回 null
解释：链表中没有环。
```



## 题解

之前甚至做过一道要求返回环起始节点的题目，这一道直接判断是否有环即可，如果要返回环起始节点，将其中一个指针放在开头同步向后直至相遇即可

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while(true) {
            if(fast == null || fast.next == null) {
                return false;
            }
            slow = slow.next;
            fast = fast.next.next;
            if(slow == fast) {
                return true;
            }
        }
    }
}
```



# 两数之和

## 题目描述

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。

你可以按任意顺序返回答案。

 

**示例 1：**

```
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
```

**示例 2：**

```
输入：nums = [3,2,4], target = 6
输出：[1,2]
```

**示例 3：**

```
输入：nums = [3,3], target = 6
输出：[0,1]
```

**提示：**

- `2 <= nums.length <= 104`
- `-109 <= nums[i] <= 109`
- `-109 <= target <= 109`
- **只会存在一个有效答案**



## 题解

解法一：暴力遍历数组，直到遇见两个元素相加为预期的结果，返回下表数组

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int n = nums.length;
        for(int i = 0; i < n; i++) {
            for(int j = i + 1; j < n; j++) {
                if(nums[i] + nums[j] == target) {
                    return new int[]{i, j};
                }
            }
        }
        return null;
    }
}
```



解法二：通过哈希表进行优化，再遍历数组的每个元素的时候，先判断map中有无target-nums[i]的键，如果有则用键取值，得到下表返回即可，如果没有将<值，下表>放入map供以后的元素取用

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < nums.length; i++) {
            Set keySet = map.keySet();
            if(keySet.contains(target - nums[i])) {
                return new int[]{map.get(target - nums[i]), i};
            }
            map.put(nums[i], i);
        }
        return null;
    }
}
```

