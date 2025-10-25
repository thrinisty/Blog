---
title: LeetCode刷题（H指数、O1时间插入删除随机获取）
published: 2025-10-24
updated: 2025-10-24
description: 'H指数、二分查找H指数、O1时间插入删除随机获取'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# H指数

## 题目描述

给你一个整数数组 `citations` ，其中 `citations[i]` 表示研究者的第 `i` 篇论文被引用的次数。计算并返回该研究者的 **`h` 指数**。

根据维基百科上 [h 指数的定义](https://baike.baidu.com/item/h-index/3991452?fr=aladdin)：`h` 代表“高引用次数” ，一名科研人员的 `h` **指数** 是指他（她）至少发表了 `h` 篇论文，并且 **至少** 有 `h` 篇论文被引用次数大于等于 `h` 。如果 `h` 有多种可能的值，**`h` 指数** 是其中最大的那个。

**示例 1：**

```
输入：citations = [3,0,6,1,5]
输出：3 
解释：给定数组表示研究者总共有 5 篇论文，每篇论文相应的被引用了 3, 0, 6, 1, 5 次。
     由于研究者有 3 篇论文每篇 至少 被引用了 3 次，其余两篇论文每篇被引用 不多于 3 次，所以她的 h 指数是 3。
```

**示例 2：**

```
输入：citations = [1,3,1]
输出：1
```



## 题解

解法一：（穷举法）从设置h为文章数量开始，向下依次判断，如果满足有h篇索引高于h的则返回当前h

```java
class Solution {
    public int hIndex(int[] citations) {
        int n = citations.length;
        for(int h = n; h > 0; h--) {
            int count = 0;
            for(int i = 0; i < n; i++) {
                if(citations[i] >= h) {
                    count++;
                }
                if(count >= h) {
                    return h;
                }
            }
        }
        return 0;
    }
}
```



解法二：先进行排序，然后从大到小遍历，看当前数是否满足大于下一个目标数，如果符合则，将h置为目标

```java
class Solution {
    public int hIndex(int[] citations) {
        int n = citations.length;
        int h = 0;
        for(int i = n - 1; i >= 0; i--) {
            if(citations[i] >= h + 1) {
                h++;
            } else {
                break;
            }
        }
        return h;
    }
}
```

解法三：如果数组是有序的后，我们还可以通过二分查找的方式对于后续的过程进行进一步的优化

```java
class Solution {
    public int hIndex(int[] citations) {
        int n = citations.length;
        if (n == 0) return 0;
        return binarySearch(citations, 0, n - 1);
    }
    
    private int binarySearch(int[] citations, int left, int right) {
        int n = citations.length;
        if (left == right) {
            return citations[right] >= n - right ? n - right : 0;
        }
        
        int mid = left + (right - left) / 2;
        if (citations[mid] >= n - mid) {
            return binarySearch(citations, left, mid);
        } else {
            return binarySearch(citations, mid + 1, right);
        }
    }
}
```



# O1时间插入删除随机获取

## 题目描述

实现`RandomizedSet` 类：

- `RandomizedSet()` 初始化 `RandomizedSet` 对象
- `bool insert(int val)` 当元素 `val` 不存在时，向集合中插入该项，并返回 `true` ；否则，返回 `false` 。
- `bool remove(int val)` 当元素 `val` 存在时，从集合中移除该项，并返回 `true` ；否则，返回 `false` 。
- `int getRandom()` 随机返回现有集合中的一项（测试用例保证调用此方法时集合中至少存在一个元素）。每个元素应该有 **相同的概率** 被返回。

你必须实现类的所有函数，并满足每个函数的 **平均** 时间复杂度为 `O(1)` 。

**示例：**

```
输入
["RandomizedSet", "insert", "remove", "insert", "getRandom", "remove", "insert", "getRandom"]
[[], [1], [2], [2], [], [1], [2], []]
输出
[null, true, false, true, 2, true, false, 2]
```



## 题解

解法一：复用一下Set集合，面试应该不会让用这种方式

```java
class RandomizedSet {
    Set<Integer> set = new HashSet<>();
    public RandomizedSet() {
        
    }
    
    public boolean insert(int val) {
        if(set.contains(val)) {
            return false;
        } else {
            set.add(val);
            return true;
        }
    }
    
    public boolean remove(int val) {
        if(set.contains(val)) {
            set.remove(val);
            return true;
        } else {
            return false;
        }
    }
    
    public int getRandom() {
        int n = set.size();
        return (int)set.toArray()[(int)(Math.random() * n)];
    }
}
```

解法二：通过ArrayList数组存储元素，由HashMap存放元素以及其对应下表，在加入和删除时用HashMap进行快速判断存在性

```java
class RandomizedSet {
    List<Integer> nums;
    Map<Integer, Integer> indices;
    Random random;
    public RandomizedSet() {
        nums = new ArrayList<>();
        indices = new HashMap<>();
        random = new Random();
    }
    
    public boolean insert(int val) {
        if(indices.containsKey(val)) {
            return false;
        }
        int index = nums.size();
        nums.add(val);
        indices.put(val, index);
        return true;
    }
    
    public boolean remove(int val) {
        if(!indices.containsKey(val)) {
            return false;
        }
        int index = indices.get(val);
        int last = nums.get(nums.size() - 1);
        nums.set(index, last);
        indices.put(last, index);
        nums.remove(nums.size() - 1);
        indices.remove(val);
        return true;
    }
    
    public int getRandom() {
        int randonNum = random.nextInt(nums.size());
        return nums.get(randonNum);
    }
}
```

