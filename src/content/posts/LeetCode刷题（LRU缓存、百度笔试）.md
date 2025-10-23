---
title: LeetCode刷题（LRU缓存、百度笔试）
published: 2025-10-19
updated: 2025-10-19
description: 'LRU缓存，平衡子串计数'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# LRU缓存

## 题目描述

请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

**示例：**

```
输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```



## 题解

解法一：直接继承于LinkedHashSet，放入时和取出时直接调用父类的方法，此外重写removeEldestEntry方法，使其在put超过容量时自动触发移除最老的数据

```java
class LRUCache extends LinkedHashMap<Integer, Integer>{
    private int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75F, true);
        this.capacity = capacity;
    }
    
    public int get(int key) {
        return super.getOrDefault(key, -1);
    }
    
    public void put(int key, int value) {
        super.put(key, value);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return this.size() > capacity;
    }
}
```



解法二：模拟底层的LinkedHashMap，通过LinkedNode存储活跃的双向链表，在对于添加，取出，修改的三种情况将节点放在头部并且添加进入HashMap中。而对于放入超出容量的情况下将尾部的节点移除并删除对应的缓存内容

```java
class LRUCache {
    class Node {
        int key;
        int value;
        Node prev;
        Node next;
        public Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    private int size;
    private int capacity;
    private Node head, tail;
    private Map<Integer, Node> cache = new HashMap<>();
    public LRUCache(int capacity) {
        this.size = 0;
        this.capacity = capacity;
        head = new Node(-1, -1);
        tail = new Node(-1, -1);
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        Node node = cache.get(key);
        if(node == null) {
            return -1;
        } 
        remove(node);
        addHead(node);
        return node.value;
    }
    
    public void put(int key, int value) {
        Node node = cache.get(key);
        if(node == null) {
            node = new Node(key, value);
            cache.put(key, node);
            addHead(node);
            size++;
            if(capacity < size) {
                Node removeNode = tail.prev;
                cache.remove(removeNode.key);
                remove(removeNode);
                size--;
            }
        }
        node.value = value;
        remove(node);
        addHead(node);
    }

    public void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    public void addHead(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next = node;
        node.next.prev = node;
    }
}
```



# 平衡子串计数

## 题目描述

给定一个由数字 `0` 到 `9` 组成的字符串，你需要统计其中有多少个长度为 `k` 的连续子串满足 **前半部分数字之和等于后半部分数字之和**。

如果 `k` 是奇数，则**中间的数字不参与计算**（仅计算左右两半的和是否相等）。

------

### **输入格式**：

- **第一行**：一个整数 `T`（`1 ≤ T ≤ 100`），表示测试用例的数量。

- 接下来 `T` 组输入

  ，每组包含：

  - **第一行**：两个整数 `n` 和 `k`（`1 ≤ k ≤ n ≤ 10^5`），其中 `n` 是字符串的长度，`k` 是子串的长度。
  - **第二行**：一个长度为 `n` 的字符串 `s`，仅由 `0`-`9` 组成。

------

### **输出格式**：

对于每个测试用例，输出一个整数，表示满足条件的子串数量。

------

### **示例**：

#### **输入 1**：

```
1
8 4
12341234
```

#### **输出 1**：

```
2
```



## 题解

解法一：暴力拆解，遍历规定长度的所有情况，可惜超时了

```java
import java.util.Scanner;
public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int length = in.nextInt();
        for (int m = 0; m < length; m++) {
            int n = in.nextInt();
            int k = in.nextInt();
            String str = in.next();
            char[] arr = str.toCharArray();
            int result = 0;
            for(int i = 0; i < n - k + 1; i++) {
                int j = i + k - 1;
                int mid = (i + j) / 2;
                int sumL = 0;
                for(int l = i; l <= mid; l++) {
                    sumL += (arr[l] - '0');
                }
                int sumR = 0;
                for(int r = mid + 1; r <= j; r++) {
                    sumR += (arr[r] - '0');
                }
                if(sumL == sumR) {
                    result++;
                }
            }
            System.out.println(result);
        }
    }
}
```

解法二：前缀和，写的时候没想起来用，回想起来觉得应该挺容易实现的

```java
package com.nwpu;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int length = in.nextInt();
        for (int m = 0; m < length; m++) {
            int n = in.nextInt();
            int k = in.nextInt();
            String str = in.next();
            char[] arr = str.toCharArray();
            int result = 0;
            int[] preSum = new int[n];
            preSum[0] = arr[0] - '0';
            for (int i = 1; i < n; i++) {
                preSum[i] = preSum[i - 1] + arr[i] - '0';
            }
            for(int i = 0; i < n - k + 1; i++) {
                int j = i + k - 1;
                int mid = i + k / 2 - 1;
                int sumL = i == 0 ? preSum[mid] : (preSum[mid] - preSum[i - 1]);
                int sumR = preSum[j] - preSum[mid];
                if(sumL == sumR) {
                    result++;
                }
            }
            System.out.println(result);
        }
    }
}
```
