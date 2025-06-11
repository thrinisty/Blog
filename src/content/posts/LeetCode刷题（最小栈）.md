---
title: LeetCode刷题（最小栈）
published: 2025-06-11
updated: 2025-06-11
description: '最小栈实现'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 最小栈

## 题目描述

设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。

实现 `MinStack` 类:

- `MinStack()` 初始化堆栈对象。
- `void push(int val)` 将元素val推入堆栈。
- `void pop()` 删除堆栈顶部的元素。
- `int top()` 获取堆栈顶部的元素。
- `int getMin()` 获取堆栈中的最小元素。

**示例 1:**

```
输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.getMin();   --> 返回 -2.
```



## 题解

这一道题是我同学的面试题目，他给我讲的时候我有些摸不清头脑，看了题解后清晰多了，主要的要点就是创建一个最小栈的结构辅助主栈中最小值的存储

这个最小栈在主栈元素入站的时判断元素是否为最小元素亦或者最小栈中没有元素，如果符合情况进行入最小栈

在pop弹出的时候，判断主栈弹出的元素是否等与最小栈栈顶的元素，满足的pop最小栈

```java
class MinStack {
    private Stack<Integer> stack;
    private Stack<Integer> minStack;
    public MinStack() {
        stack = new Stack<>();
        minStack = new Stack<>();
    }
    
    public void push(int val) {
        stack.push(val);
        if(minStack.isEmpty() || val <= minStack.peek()) {
            minStack.push(val);
        }
    }
    
    public void pop() {
        int num = stack.pop();
        if(num == minStack.peek()) {
            minStack.pop();
        }
    }
    
    public int top() {
        return stack.peek();
    }
    
    public int getMin() {
        return minStack.peek();
    }
}
```

