---
title: LeetCode刷题（数据流中的第K大元素、单词替换）
published: 2025-12-17
updated: 2025-12-17
description: '数据流中的第K大元素、单词替换'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 数据流中的第K大元素

## 题目描述

设计一个找到数据流中第 `k` 大元素的类（class）。注意是排序后的第 `k` 大元素，不是第 `k` 个不同的元素。

请实现 `KthLargest` 类：

- `KthLargest(int k, int[] nums)` 使用整数 `k` 和整数流 `nums` 初始化对象。
- `int add(int val)` 将 `val` 插入数据流 `nums` 后，返回当前数据流中第 `k` 大的元素。

**示例：**

```
输入：
["KthLargest", "add", "add", "add", "add", "add"]
[[3, [4, 5, 8, 2]], [3], [5], [10], [9], [4]]
输出：
[null, 4, 5, 5, 8, 8]

解释：
KthLargest kthLargest = new KthLargest(3, [4, 5, 8, 2]);
kthLargest.add(3);   // return 4
kthLargest.add(5);   // return 5
kthLargest.add(10);  // return 5
kthLargest.add(9);   // return 8
kthLargest.add(4);   // return 8
```

## 题解

```java
class KthLargest {
    PriorityQueue<Integer> heap;
    int k;

    public KthLargest(int k, int[] nums) {
        this.k = k;
        heap = new PriorityQueue<>();
        for(int num : nums) {
            heap.offer(num);
            if(heap.size() > k) {
                heap.poll();
            }
        }
    }
    
    public int add(int val) {
        heap.offer(val);
        if(heap.size() > k) {
            heap.poll();
        }
        return heap.peek();
    }
}
```



# 单词替换

## 题目描述

在英语中，有一个叫做 `词根(root)` 的概念，它可以跟着其他一些词组成另一个较长的单词——我们称这个词为 `继承词(successor)`。例如，词根`an`，跟随着单词 `other`(其他)，可以形成新的单词 `another`(另一个)。

现在，给定一个由许多词根组成的词典和一个句子，需要将句子中的所有`继承词`用`词根`替换掉。如果`继承词`有许多可以形成它的`词根`，则用最短的词根替换它。

需要输出替换之后的句子。

**示例 1：**

```
输入：dictionary = ["cat","bat","rat"], sentence = "the cattle was rattled by the battery"
输出："the cat was rat by the bat"
```

**示例 2：**

```
输入：dictionary = ["a","b","c"], sentence = "aadsfasf absbs bbab cadsfafs"
输出："a a b c"
```

**示例 3：**

```
输入：dictionary = ["a", "aa", "aaa", "aaaa"], sentence = "a aa a aaaa aaa aaa aaa aaaaaa bbb baba ababa"
输出："a a a a a a a a bbb baba a"
```

**示例 4：**

```
输入：dictionary = ["catt","cat","bat","rat"], sentence = "the cattle was rattled by the battery"
输出："the cat was rat by the bat"
```

**示例 5：**

```
输入：dictionary = ["ac","ab"], sentence = "it is abnormal that this solution is accepted"
输出："it is ab that this solution is ac"
```



## 题解

依旧用前缀树解决，但是不存储isEnd了，而是存储到这里的前缀单词，提供放入前缀的函数，以及判断单词是否在树中有前缀，对于每个词进行判断，之后构造结果

get就是依次判断有没有前缀，如果中间看到了后续的字符为null则没有对应前缀，如果遍历到了一个节点发现有对应的前缀即word不为null，则返回word前缀

```java
class Solution {
    class Node {
        String word = null;
        Node[] next = new Node[26];
    }

    Node root = new Node();
    public String replaceWords(List<String> dictionary, String sentence) {
        for(String str : dictionary) {
            insert(str);
        }
        String[] arr = sentence.split(" ");
        for(int i = 0; i < arr.length; i++) {
            arr[i] = get(arr[i]) == null ? arr[i] : get(arr[i]);
        } 
        StringBuilder result = new StringBuilder();
        for(int i = 0; i < arr.length; i++) {
            String str = arr[i];
            result.append(str + " ");
        }
        result.deleteCharAt(result.length() - 1);
        return result.toString();
    }

    void insert(String word) {
        Node p = root;
        for(char c : word.toCharArray()) {
            int index = c - 'a';
            if(p.next[index] == null) {
                p.next[index] = new Node();
            }
            p = p.next[index];
        }
        p.word = word;
    }

    String get(String word) {
        Node p = root;
        for(char c : word.toCharArray()) {
            int index = c - 'a';
            if(p.next[index] == null) {
                return null;
            }
            p = p.next[index];
            if(p.word != null) {
                break;
            }
        }
        return p.word;
    }
}
```

