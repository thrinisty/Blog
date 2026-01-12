---
title: LeetCode刷题（打开转盘锁）
published: 2026-01-10
updated: 2026-01-10
description: '打开转盘锁'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 打开转盘锁

## 题目描述

一个密码锁由 4 个环形拨轮组成，每个拨轮都有 10 个数字： `'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'` 。每个拨轮可以自由旋转：例如把 `'9'` 变为 `'0'`，`'0'` 变为 `'9'` 。每次旋转都只能旋转一个拨轮的一位数字。

锁的初始数字为 `'0000'` ，一个代表四个拨轮的数字的字符串。

列表 `deadends` 包含了一组死亡数字，一旦拨轮的数字和列表里的任何一个元素相同，这个锁将会被永久锁定，无法再被旋转。

字符串 `target` 代表可以解锁的数字，请给出解锁需要的最小旋转次数，如果无论如何不能解锁，返回 `-1` 。

**示例 1：**

```
输入：deadends = ["0201","0101","0102","1212","2002"], target = "0202"
输出：6
解释：
可能的移动序列为 "0000" -> "1000" -> "1100" -> "1200" -> "1201" -> "1202" -> "0202"。
注意 "0000" -> "0001" -> "0002" -> "0102" -> "0202" 这样的序列是不能解锁的，因为当拨动到 "0102" 时这个锁就会被锁定。
```

**示例 2：**

```
输入: deadends = ["8888"], target = "0009"
输出：1
解释：
把最后一位反向旋转一次即可 "0000" -> "0009"。
```

**示例 3：**

```
输入: deadends = ["8887","8889","8878","8898","8788","8988","7888","9888"], target = "8888"
输出：-1
解释：
无法旋转到目标数字且不被锁定。
```

**示例 4：**

```
输入: deadends = ["0000"], target = "8888"
输出：-1
```



## 题解

提供一个get方法提供所有拨动一个轮可以到达的status，用广搜来找到最小的步数能到达目标

```java
class Solution {
    public int openLock(String[] deadends, String target) {
        if(target.equals("0000")) return 0;
        Set<String> dead = new HashSet<>();
        for(String deadend : deadends) {
            dead.add(deadend);
        }
        if(dead.contains("0000")) {
            return -1;
        }
        int step = 0;
        Queue<String> queue = new LinkedList<>();
        queue.offer("0000");
        Set<String> visited = new HashSet<>();
        queue.offer("0000");
        while(!queue.isEmpty()) {
            step++;
            int size = queue.size();
            for(int i = 0; i < size; i++) {
                String status = queue.poll();
                for(String nextStatus : get(status)) {
                    if(!visited.contains(nextStatus) && !dead.contains(nextStatus)) {
                        if(target.equals(nextStatus)) {
                            return step;
                        }
                        queue.offer(nextStatus);
                        visited.add(nextStatus);
                    }
                }
            }
        }
        return -1;
    }

    public char prev(char num) {
        return num == '0' ? '9' : (char)(num - 1);
    }

    public char next(char num) {
        return num == '9' ? '0' : (char)(num + 1);
    }

    public List<String> get(String status) {
        List<String> result = new ArrayList<>();
        char[] array = status.toCharArray();
        for(int i = 0; i < 4; i++) {
            char num = array[i];
            array[i] = prev(num);
            result.add(new String(array));
            array[i] = next(num);
            result.add(new String(array));
            array[i] = num;
        }
        return result;
    }
}
```

