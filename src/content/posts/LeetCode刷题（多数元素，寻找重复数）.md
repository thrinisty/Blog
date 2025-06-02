---
title: LeetCode刷题（多数元素，寻找重复数）
published: 2025-06-02
updated: 2025-06-02
description: '多数元素，寻找重复数'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 多数元素

## 题目描述

给定一个大小为 `n` 的数组 `nums` ，返回其中的多数元素。多数元素是指在数组中出现次数 **大于** `⌊ n/2 ⌋` 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

 

**示例 1：**

```
输入：nums = [3,2,3]
输出：3
```

**示例 2：**

```
输入：nums = [2,2,1,1,1,2,2]
输出：2
```



## 题解

解法一：先对数组进行排序，返回数组中间的数字（无论是全在左侧还是全在右侧亦或者是其他情况，中间的数字都是大于总数一半的数字）

```java
class Solution {
    public int majorityElement(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length / 2];
    }
}
```

解法二：摩尔投票

推论一： 若记 众数 的票数为 +1 ，非众数 的票数为 −1 ，则一定有所有数字的 票数和 >0 。

推论二： 若数组的前 a 个数字的 票数和 =0 ，则 数组剩余 (n−a) 个数字的 票数和一定仍 >0 ，即后 (n−a) 个数字的 众数仍为 x 

```java
class Solution {
    public int majorityElement(int[] nums) {
        int count = 0;
        Integer candidate = null;
        for(int num : nums) {
            if(count == 0) {
                candidate = num;
            }
            count += (num == candidate) ? 1 : -1;
        }
        return candidate;
    }
}
```

解法三：利用一个Map集合将所有的数字放入其中，value记录数字数量，再次便利map集合，当一个key对应的数值部分大于数组一半的时候返回这个key作为结果

```java
class Solution {
    public int majorityElement(int[] nums) {
        Map<Integer, Integer> map = new HashMap<>();
        for(int num : nums) {
            int i = map.getOrDefault(num, 0);
            i++;
            map.put(num, i);
        }
        int target = nums.length / 2;
        Set<Integer> set = map.keySet();
        for(Integer key : set) {
            if(map.get(key) > target) {
                return key;
            }
        }
        return 0;
    }
}
```



# 寻找重复数

## 题目描述

给定一个包含 `n + 1` 个整数的数组 `nums` ，其数字都在 `[1, n]` 范围内（包括 `1` 和 `n`），可知至少存在一个重复的整数。

假设 `nums` 只有 **一个重复的整数** ，返回 **这个重复的数** 。

你设计的解决方案必须 **不修改** 数组 `nums` 且只用常量级 `O(1)` 的额外空间。

 

**示例 1：**

```
输入：nums = [1,3,4,2,2]
输出：2
```

**示例 2：**

```
输入：nums = [3,1,3,4,2]
输出：3
```

**示例 3 :**

```
输入：nums = [3,3,3,3,3]
输出：3
```



## 题解

用数组模拟链表，当有重复的数的时候，出现环，通过快慢指针找出环的入口即为重复的数字

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int slow = 0;
        int fast = 0;
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while(slow != fast);

        slow = 0;
        while(slow != fast) {
            slow = nums[slow];
            fast = nums[fast];
        }
        return slow;
    }
}
```

第一次接触环形链表的判断，以及找到环的入口，不太理解，先背下来

第一部分：使得快慢指针指向起始位置，开始移动，直到指向的节点重复

第二部分：将慢指针指向起始位置，快慢指针依次移动一格，重复直到指向相同节点，该节点为环开始的节点



# 环形链表

额外加的：主要是上一题没有该题的基础

## 题目描述

定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 *如果链表无环，则返回 `null`。*

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

**不允许修改** 链表。



 

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

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while(true) {
            if(fast == null || fast.next == null) {
                return null;
            }
            slow = slow.next;
            fast = fast.next.next;
            if(slow == fast) {
                break;
            }
        }
        slow = head;
        while(slow != fast) {
            slow = slow.next;
            fast = fast.next;
        }
        return slow;
    }
}
```



