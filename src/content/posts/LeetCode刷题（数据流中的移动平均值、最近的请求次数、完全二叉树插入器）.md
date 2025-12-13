---
title: LeetCode刷题（数据流中的移动平均值、最近的请求次数、完全二叉树插入器）
published: 2025-12-13
updated: 2025-12-13
description: '数据流中的移动平均值、最近的请求次数、完全二叉树插入器'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 数据流中的移动平均值

## 题目描述

给定一个整数数据流和一个窗口大小，根据该滑动窗口的大小，计算滑动窗口里所有数字的平均值。

实现 `MovingAverage` 类：

- `MovingAverage(int size)` 用窗口大小 `size` 初始化对象。
- `double next(int val)` 成员函数 `next` 每次调用的时候都会往滑动窗口增加一个整数，请计算并返回数据流中最后 `size` 个值的移动平均值，即滑动窗口里所有数字的平均值。

**示例：**

```
输入：
inputs = ["MovingAverage", "next", "next", "next", "next"]
inputs = [[3], [1], [10], [3], [5]]
输出：
[null, 1.0, 5.5, 4.66667, 6.0]

解释：
MovingAverage movingAverage = new MovingAverage(3);
movingAverage.next(1); // 返回 1.0 = 1 / 1
movingAverage.next(10); // 返回 5.5 = (1 + 10) / 2
movingAverage.next(3); // 返回 4.66667 = (1 + 10 + 3) / 3
movingAverage.next(5); // 返回 6.0 = (10 + 3 + 5) / 3
```



## 题解

运用队列，以及维护一个count和加入元素时进行判断删除即可

```java
class MovingAverage {
    private int size;
    private double count;
    private Queue<Integer> queue;
    /** Initialize your data structure here. */
    public MovingAverage(int size) {
        this.count = 0;
        this.size = size;
        this.queue = new LinkedList<>();
    }
    
    public double next(int val) {
        if(size == queue.size()) {
            double num = queue.poll();
            count -= num;
        }
        queue.offer(val);
        count += val;
        return count / queue.size();
    }
}
```



# 最近的请求次数

## 题目描述

写一个 `RecentCounter` 类来计算特定时间范围内最近的请求。

请实现 `RecentCounter` 类：

- `RecentCounter()` 初始化计数器，请求数为 0 。
- `int ping(int t)` 在时间 `t` 添加一个新请求，其中 `t` 表示以毫秒为单位的某个时间，并返回过去 `3000` 毫秒内发生的所有请求数（包括新请求）。确切地说，返回在 `[t-3000, t]` 内发生的请求数。

**保证** 每次对 `ping` 的调用都使用比之前更大的 `t` 值。

**示例：**

```
输入：
inputs = ["RecentCounter", "ping", "ping", "ping", "ping"]
inputs = [[], [1], [100], [3001], [3002]]
输出：
[null, 1, 2, 3, 3]

解释：
RecentCounter recentCounter = new RecentCounter();
recentCounter.ping(1);     // requests = [1]，范围是 [-2999,1]，返回 1
recentCounter.ping(100);   // requests = [1, 100]，范围是 [-2900,100]，返回 2
recentCounter.ping(3001);  // requests = [1, 100, 3001]，范围是 [1,3001]，返回 3
recentCounter.ping(3002);  // requests = [1, 100, 3001, 3002]，范围是 [2,3002]，返回 3
```



## 题解

构造队列，在放入之后移除不在范围内的元素，最后返回队列大小即可，因为队列中按照时间放入，最近的元素是最后一个时间

```java
class RecentCounter {
    Queue<Integer> queue;

    public RecentCounter() {
        queue = new LinkedList<>();
    }
    
    public int ping(int t) {
        queue.offer(t);
        while(queue.peek() < t - 3000) {
            queue.poll();
        }
        return queue.size();
    }
}
```



# 完全二叉树插入器

## 题目描述

完全二叉树是每一层（除最后一层外）都是完全填充（即，节点数达到最大，第 `n` 层有 `2n-1` 个节点）的，并且所有的节点都尽可能地集中在左侧。

设计一个用完全二叉树初始化的数据结构 `CBTInserter`，它支持以下几种操作：

- `CBTInserter(TreeNode root)` 使用根节点为 `root` 的给定树初始化该数据结构；
- `CBTInserter.insert(int v)` 向树中插入一个新节点，节点类型为 `TreeNode`，值为 `v` 。使树保持完全二叉树的状态，**并返回插入的新节点的父节点的值**；
- `CBTInserter.get_root()` 将返回树的根节点。

**示例 1：**

```
输入：inputs = ["CBTInserter","insert","get_root"], inputs = [[[1]],[2],[]]
输出：[null,1,[1,2]]
```

**示例 2：**

```
输入：inputs = ["CBTInserter","insert","insert","get_root"], inputs = [[[1,2,3,4,5,6]],[7],[8],[]]
输出：[null,3,4,[1,2,3,4,5,6,7,8]]
```



## 题解

构造一个候选者队列，初始化层序遍历，将从层序遍历辅助队列中节点取出时判断其有可以插入的位置，就将该节点放入候选者队列中，之后加入结点的时候就将子节点放入第一个出队的节点的子节点位置（先左再右），如果放满了就将将这个候选者出队，最后将新放入的节点放入候选者队列中

```java
class CBTInserter {
    Queue<TreeNode> candidate;
    TreeNode root;

    public CBTInserter(TreeNode root) {
        this.root = root;
        this.candidate = new LinkedList<>();

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if(node.left != null) {
                queue.offer(node.left);
            }
            if(node.right != null) {
                queue.offer(node.right);
            }
            if(node.left == null || node.right == null) {
                candidate.offer(node);
            }
        }

    }
    
    public int insert(int v) {
        TreeNode child = new TreeNode(v);
        TreeNode node = candidate.peek();
        if(node.left == null) {
            node.left = child;
        } else {
            node.right = child;
            candidate.poll();
        }
        candidate.offer(child);
        return node.val;
    }
    
    public TreeNode get_root() {
        return root;
    }
}
```

