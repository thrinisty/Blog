---
title: LeetCode刷题（数组的相对排序、组合总数Ⅱ）
published: 2025-12-24
updated: 2025-12-24
description: '数组的相对排序、组合总数Ⅱ'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 数组的相对排序

## 题目描述

给定两个数组，`arr1` 和 `arr2`，

- `arr2` 中的元素各不相同
- `arr2` 中的每个元素都出现在 `arr1` 中

对 `arr1` 中的元素进行排序，使 `arr1` 中项的相对顺序和 `arr2` 中的相对顺序相同。未在 `arr2` 中出现过的元素需要按照升序放在 `arr1` 的末尾。

**示例：**

```
输入：arr1 = [2,3,1,3,2,4,6,7,9,2,19], arr2 = [2,1,4,3,9,6]
输出：[2,2,2,1,4,3,3,9,6,7,19]
```

**提示：**

- `1 <= arr1.length, arr2.length <= 1000`
- `0 <= arr1[i], arr2[i] <= 1000`
- `arr2` 中的元素 `arr2[i]` 各不相同
- `arr2` 中的每个元素 `arr2[i]` 都出现在 `arr1` 中



## 题解

拆成两个部分分别排序即可，map排序的规则自定义

```java
class Solution {
    public int[] relativeSortArray(int[] arr1, int[] arr2) {
        int n = arr1.length;
        int[] result = new int[n];
        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < arr2.length; i++) {
            map.put(arr2[i], i);
        }
        List<Integer> list1 = new LinkedList<>(); 
        List<Integer> list2 = new LinkedList<>(); 
        for(int num : arr1) {
            if(map.containsKey(num)) {
                list1.add(num);
            } else {
                list2.add(num);
            }
        }
        Collections.sort(list1, (a, b) -> map.get(a) - map.get(b));
        Collections.sort(list2);
        int index = 0;
        for(int i = 0; i < list1.size(); i++) {
            result[index++] = list1.get(i);
        }
        for(int i = 0; i < list2.size(); i++) {
            result[index++] = list2.get(i);
        }
        return result;
    }
}
```



# 组合总数Ⅱ

## 题目描述

给定一个可能有重复数字的整数数组 `candidates` 和一个目标数 `target` ，找出 `candidates` 中所有可以使数字和为 `target` 的组合。

`candidates` 中的每个数字在每个组合中只能使用一次，解集不能包含重复的组合。 

**示例 1：**

```
输入：candidates = [10,1,2,7,6,1,5], target = 8
输出：
[
[1,1,6],
[1,2,5],
[1,7],
[2,6]
]
```

**示例 2：**

```
输入：candidates = [2,5,2,1,2], target = 5
输出：
[
[1,2,2],
[5]
]
```



## 题解

先进行数组排序，循环判断添加当前的元素是否大于target如果大于则后续也能找到更小的可能直接结束，否则尝试添加，如果当前数和前一个一致则跳过，避免重复，剩下的思路与组合总数一致

```java
class Solution {
    List<List<Integer>> result = new LinkedList<>();
    List<Integer> list = new LinkedList<>();

    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        dfs(candidates, target, 0);
        return result;
    }

    public void dfs(int[] candidates, int target, int index) {
        if(target == 0) {
            result.add(new LinkedList<>(list));
            return;
        }
        for(int i = index; i < candidates.length; i++) {
            if(candidates[i] > target) {
                break;
            }
            if(i > index && candidates[i] == candidates[i - 1]) {
                continue;
            }
            list.add(candidates[i]);
            dfs(candidates, target - candidates[i], i + 1);
            list.remove(list.size() - 1);
        }
    }
}
```

