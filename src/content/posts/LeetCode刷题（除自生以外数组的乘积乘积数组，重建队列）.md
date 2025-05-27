---
title: LeetCode刷题（乘积数组，重建队列）
published: 2025-05-23
updated: 2025-05-23
description: '除自生以外数组的乘积，根据身高重建队列'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 除自生以外数组的乘积

## 题目描述

给你一个整数数组 `nums`，返回 数组 `answer` ，其中 `answer[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积 。

题目数据 **保证** 数组 `nums`之中任意元素的全部前缀元素和后缀的乘积都在 **32 位** 整数范围内。

请 **不要使用除法，**且在 `O(n)` 时间复杂度内完成此题。

**示例 1:**

```
输入: nums = [1,2,3,4]
输出: [24,12,8,6]
```

**示例 2:**

```
输入: nums = [-1,1,0,-3,3]
输出: [0,0,9,0,0]
```



## 题解

用传统的求数组乘积的方法，在数组中存在0的时候失效，因为使用除法/0会导致异常，我们构造两个辅助数组，分别计算出该位置左右两侧所有的数相乘结果，在使用两个数组逐个相乘即可得出答案

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] L = new int[n];
        int[] R = new int[n];
        int[] result = new int[n];
        L[0] = 1;
        for (int i = 1; i < n; i++) {
            L[i] = L[i - 1] * nums[i - 1];
        }
        R[n - 1] = 1;
        for (int i = n - 2; i >= 0; i--) {
            R[i] = R[i + 1] * nums[i + 1]; 
        }
        for(int i = 0; i < n; i++) {
            result[i] = L[i] * R[i];
        }
        return result;
    }
}
```

还可以进一步改善，不创建辅助数组，直接在结果数组中进行操作

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] result = new int[n];

        //用结果数组存储每个数左侧的乘积
        result[0] = 1; 
        for (int i = 1; i < n; i++) {
            result[i] = result[i - 1] * nums[i - 1];
        }

        //右侧的乘积结果使用遍历R表示，初始为1
        int R = 1;
        for (int i = n - 1; i >= 0; i--) {
            result[i] *= R;
            R *= nums[i];
        }
        return result;
    }
}
```



# 根据身高重建队列

## 题目描述

假设有打乱顺序的一群人站成一个队列，数组 `people` 表示队列中一些人的属性（不一定按顺序）。每个 `people[i] = [hi, ki]` 表示第 `i` 个人的身高为 `hi` ，前面 **正好** 有 `ki` 个身高大于或等于 `hi` 的人。

请你重新构造并返回输入数组 `people` 所表示的队列。返回的队列应该格式化为数组 `queue` ，其中 `queue[j] = [hj, kj]` 是队列中第 `j` 个人的属性（`queue[0]` 是排在队列前面的人）。

**示例 1：**

```
输入：people = [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]
输出：[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]
解释：
编号为 0 的人身高为 5 ，没有身高更高或者相同的人排在他前面。
编号为 1 的人身高为 7 ，没有身高更高或者相同的人排在他前面。
编号为 2 的人身高为 5 ，有 2 个身高更高或者相同的人排在他前面，即编号为 0 和 1 的人。
编号为 3 的人身高为 6 ，有 1 个身高更高或者相同的人排在他前面，即编号为 1 的人。
编号为 4 的人身高为 4 ，有 4 个身高更高或者相同的人排在他前面，即编号为 0、1、2、3 的人。
编号为 5 的人身高为 7 ，有 1 个身高更高或者相同的人排在他前面，即编号为 1 的人。
因此 [[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]] 是重新构造后的队列。
```



## 题解

先运用sort传入内部类，进行定制排序（将升高按照从大到小排列，相同时前面人数多的放在后面），实现后面的人放入集合不影响前面的队形

这样的话按照顺序遍历int二维数组，将每个一维数组按照前面存在的人数进行放入（以及放入的人都比这个人要高）即可实现，最后将List集合转化为数组返回

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        Arrays.sort(people, new Comparator<int[]>() {
            public int compare(int[] people1, int[] people2) {
                if(people1[0] != people2[0]) {
                    return people2[0] - people1[0];
                } 
                return people1[1] - people2[1];
            }
        });
        List<int[]> ans = new ArrayList<>();
        for(int [] person : people) {
            ans.add(person[1], person);
        }
       ans.toArray(people);
        return people;
    }
}
```

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        Arrays.sort(people, new Comparator<int[]>() {
            public int compare(int[] person1, int[] person2) {
                if(person1[0] != person2[0]) {
                    return person2[0] - person1[0];
                }
                return person1[1] - person2[1];
            }
        });
        List<int[]> ans = new ArrayList<>();
        for(int[] person : people) {
            ans.add(person[1], person);
        }
        int n = ans.size();
        for(int i = 0; i < n; i++) {
            people[i] = ans.get(i);
        }
        return people;
    }
}
```

