---
title: LeetCode刷题（键值映射、数组中两个数的最大异或值）
published: 2025-12-20
updated: 2025-12-20
description: '数组中两个数的最大异或值'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 键值映射

## 题目描述

实现一个 `MapSum` 类，支持两个方法，`insert` 和 `sum`：

- `MapSum()` 初始化 `MapSum` 对象
- `void insert(String key, int val)` 插入 `key-val` 键值对，字符串表示键 `key` ，整数表示值 `val` 。如果键 `key` 已经存在，那么原来的键值对将被替代成新的键值对。
- `int sum(string prefix)` 返回所有以该前缀 `prefix` 开头的键 `key` 的值的总和。

**示例：**

```
输入：
inputs = ["MapSum", "insert", "sum", "insert", "sum"]
inputs = [[], ["apple", 3], ["ap"], ["app", 2], ["ap"]]
输出：
[null, null, 3, null, 5]

解释：
MapSum mapSum = new MapSum();
mapSum.insert("apple", 3);  
mapSum.sum("ap");           // return 3 (apple = 3)
mapSum.insert("app", 2);    
mapSum.sum("ap");           // return 5 (apple + app = 3 + 2 = 5)
```



## 题解

解法一：sum暴力扫描，对于每个符合条件的key使得结果+value

```java
class MapSum {
    Map<String, Integer> map = new HashMap<>();
    
    public void insert(String key, int val) {
        map.put(key, val);
    }
    
    public int sum(String prefix) {
        int result = 0;
        for(String key : map.keySet()) {
            if(key.startsWith(prefix)) {
                result += map.get(key);
            }
        }
        return result;
    }   
}
```

解法二：在insert构造的时候将所有的前缀赋予对应的值，重复的+value，在查找的时候直接差对应得前缀Map即可

```java
class MapSum {
    Map<String, Integer> map;
    Map<String, Integer> prefixmap;
    /** Initialize your data structure here. */
    public MapSum() {
        map = new HashMap<>();
        prefixmap = new HashMap<>();
    }
    
    public void insert(String key, int val) {
        int del = val - map.getOrDefault(key, 0);
        map.put(key, val);
        for(int i = 0; i < key.length(); i++) {
            String preStr = key.substring(0, i + 1);
            prefixmap.put(preStr, prefixmap.getOrDefault(preStr, 0) + del);
        }
    }
    
    public int sum(String prefix) {
        return prefixmap.getOrDefault(prefix, 0);
    }
}
```

解法三：通过字典树进行求解

在插入的时候给每个位置加上del，在取出的时候，遍历到最后取出对应value

```java
class MapSum {
    class Node {
        int value = 0;
        Node[] next = new Node[26];
    }

    Node root;
    Map<String, Integer> map;
    public MapSum() {
        root = new Node();
        map = new HashMap<>();
    }
    
    public void insert(String key, int val) {
        int del = val - map.getOrDefault(key, 0);
        map.put(key, val);
        Node p = root;
        for(char c : key.toCharArray()) {
            int index = c - 'a';
            if(p.next[index] == null) {
                p.next[index] = new Node();
            }
            p = p.next[index];
            p.value += del;
        }
    }
    
    public int sum(String prefix) {
        Node p = root;
        for(char c : prefix.toCharArray()) {
            int index = c - 'a';
            if(p.next[index] == null) {
                return 0;
            }
            p = p.next[index];
        }
        return p.value;
    }
}
```



# 数组中两个数的最大异或值

## 题目描述

给定一个整数数组 `nums` ，返回 `nums[i] XOR nums[j]` 的最大运算结果，其中 `0 ≤ i ≤ j < n` 。

**示例 1：**

```
输入：nums = [3,10,5,25,2,8]
输出：28
解释：最大运算结果是 5 XOR 25 = 28.
```

**示例 2：**

```
输入：nums = [0]
输出：0
```

**示例 3：**

```
输入：nums = [2,4]
输出：6
```

**示例 4：**

```
输入：nums = [8,10,2]
输出：10
```

**示例 5：**

```
输入：nums = [14,70,53,83,49,91,36,80,92,51,66,70]
输出：127
```



## 题解

解法一：暴力求解（超时40/41）

```java
class Solution {
    public int findMaximumXOR(int[] nums) {
        int max = 0;
        int n = nums.length;
        for(int i = 0; i < n; i++) {
            for(int j = i + 1; j < n; j++) {
                max = Math.max(max, nums[i] ^ nums[j]);
            }
        }
        return max;
    }
}
```

解法二：通过字典树构造每个数字的字典树，之后通过逐个从高位取出一个位，如果有相反的位优先取相反的位，没有再取同位，构造出一个理想的要要异或的数，最后异或

```java
class Solution {
    class Node {
        Node[] next = new Node[2];
    }

    Node root = new Node();
    public int findMaximumXOR(int[] nums) {
        for(int num : nums) {
            build(num);
        }
        int max = 0;
        for(int num : nums) {
            max = Math.max(max, search(num));
        }
        return max;
    }

    public void build(int num) {
        Node p = root;
        for(int i = 30; i >= 0; i--) {
            int index = (num >> i) & 1;
            if(p.next[index] == null) {
                p.next[index] = new Node();
            }
            p = p.next[index];
        }
    }

    public int search(int num) {
        Node p = root;
        int target = 0;
        for(int i = 30; i >= 0; i--) {
            int current = (num >> i) & 1;
            int reverse = current == 1 ? 0 : 1;
            if(p.next[reverse] != null) {
                p = p.next[reverse];
                target = target * 2 + reverse;
            } else {
                p = p.next[current];
                target = target * 2 + current;
            }
        }
        return num ^ target;
    }
}
```

