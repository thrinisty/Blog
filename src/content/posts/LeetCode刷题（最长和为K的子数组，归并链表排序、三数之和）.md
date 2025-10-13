---
title: LeetCode刷题（最长和为K的子数组，归并链表排序、三数之和）
published: 2025-10-09
updated: 2025-10-09
description: '最长和为K的子数组，归并链表排序、三数之和'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 最长和为K的子数组

## 题目描述

和之前的和为K的子数组相似，要求返回最长的子数组大小



## 题解

还是用前缀和的方式进行求解，只不过Map后一个元素存储的不再是前面可以组合成为某一个值的可能数量了，而是存储第一个满足该前缀和的下表（后续存在不更新）

当前面存在目标前缀和的时候，用当前遍历的元素下减去前缀和所对应的下表，用这个结果迭代result

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        if(nums == null || nums.length == 0) {
            return 0;
        }
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, -1);
        int preSum = 0;
        int result = 0;
        for(int i = 0; i < nums.length; i++) {
            preSum += nums[i];
            int target = preSum - k;
            if(map.containsKey(target)) {
                result = Math.max(result, i - map.get(target));
            }
            if(!map.containsKey(preSum)) {
                map.put(preSum, i);
            }
        }
        return result;
    }
}
```



# 归并链表排序

## 题目描述

给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表**

## 题解

之前我们做过这一道题目，用的解法是递归的将头节点插入后续已排序的链表中，这样的时间复杂度为O(n2)，在模拟面试的题目中这种解法超时，我们采用更加快速的归并排序方式进行求解

用到回文链表中的returnHalf方法返回中间节点，在主方法中将前后断开分为两个两个链表，分别排序后用实现的mergeList方法合并

```java
class Solution {
    public ListNode sortList(ListNode head) {
        if(head == null || head.next == null) {
            return head;
        }
        ListNode p = returnHalf(head);
        ListNode midNode = p.next;
        p.next = null;
        ListNode left = sortList(head);
        ListNode right = sortList(midNode);
        return mergeList(left, right);
    }

    public ListNode returnHalf(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while(fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }

    public ListNode mergeList(ListNode n1, ListNode n2) {
        ListNode result = new ListNode();
        ListNode p = result;
        while(n1 != null && n2 != null) {
            if(n1.val < n2.val) {
                p.next = n1;
                n1 = n1.next;
                p = p.next;
            } else {
                p.next = n2;
                n2 = n2.next;
                p = p.next;
            }
        }
        p.next = n1 == null ? n2 : n1;
        return result.next;
    }
}
```



# 三数之和

## 题目描述

给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

**示例 1：**

```
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
解释：
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。
不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。
注意，输出的顺序和三元组的顺序并不重要。
```

**示例 2：**

```
输入：nums = [0,1,1]
输出：[]
解释：唯一可能的三元组和不为 0 。
```

**示例 3：**

```
输入：nums = [0,0,0]
输出：[[0,0,0]]
解释：唯一可能的三元组和为 0 。
```



## 题解

将整个链表从小到大排序，依次选取第一个数，如果该数大于0则代表后续不可能有和为0的情况的发生

用L和R标记选取的第一个数的下一个位置和最后一个位置，while循环遍历后续的地方，如果三数和为0则将其加入到结果集中并选取下一个非重复的大值和小值，如果结果小于0代表L选取的数字不够大L++，如果大于0则表示R选取的数字过大，R--选取更小的数字。

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(nums);
        int n = nums.length;
        if(nums == null || n < 3) {
            return result;
        }
        for(int i = 0; i < n - 2; i++) {
            if(nums[i] > 0) {
                break;
            }
            if(i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            int L = i + 1;
            int R = n - 1;
            while(L < R) {
                int count = nums[i] + nums[L] + nums[R];
                if(count == 0) {
                    result.add(Arrays.asList(nums[i], nums[L], nums[R]));
                    while (L < R && nums[L] == nums[L + 1]) L++;
                    while (L < R && nums[R] == nums[R - 1]) R--;
                    L++;
                    R--;
                } else if(count < 0) {            
                    L++;
                } else {
                    R--;
                }
            }
        }
        return result;
    }
}
```

