---
title: LeetCode刷题（正则表达式、课程表）
published: 2025-10-27
updated: 2025-10-27
description: '正则表达式匹配、课程表'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 正则表达式匹配

## 题目描述

给你一个字符串 `s` 和一个字符规律 `p`，请你来实现一个支持 `'.'` 和 `'*'` 的正则表达式匹配。

- `'.'` 匹配任意单个字符
- `'*'` 匹配零个或多个前面的那一个元素

所谓匹配，是要涵盖 **整个** 字符串 `s` 的，而不是部分字符串。

**示例 1：**

```
输入：s = "aa", p = "a"
输出：false
解释："a" 无法匹配 "aa" 整个字符串。
```

**示例 2:**

```
输入：s = "aa", p = "a*"
输出：true
解释：因为 '*' 代表可以匹配零个或多个前面的那一个元素, 在这里前面的元素就是 'a'。因此，字符串 "aa" 可被视为 'a' 重复了一次。
```

**示例 3：**

```
输入：s = "ab", p = ".*"
输出：true
解释：".*" 表示可匹配零个或多个（'*'）任意字符（'.'）。
```



## 题解

看不懂，先背下来，之后再想

```java
class Solution {
    public boolean isMatch(String s, String p) {
        int m = s.length();
        int n = p.length();
        boolean[][] dp = new boolean[m + 1][n + 1];
        dp[0][0] = true;
        for(int i = 0; i <= m; i++) {
            for(int j = 1; j <= n; j++) {
                if(p.charAt(j - 1) == '*') {
                    dp[i][j] = dp[i][j - 2];
                    if(matches(s, p, i, j - 1)) {
                        dp[i][j] = dp[i][j] || dp[i - 1][j];
                    }
                } else {
                    if(matches(s, p, i, j)) {
                        dp[i][j] = dp[i - 1][j - 1];
                    }
                }
            }
        }
        return dp[m][n];
    }

    public boolean matches(String s, String p, int i, int j) {
        if(i == 0) {
            return false;
        }
        if(p.charAt(j - 1) == '.') {
            return true;
        }
        return s.charAt(i - 1) == p.charAt(j - 1);
    }
}
```



# 课程表

## 题目描述

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。

在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程 `bi` 。

- 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。

请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。

**示例 1：**

```
输入：numCourses = 2, prerequisites = [[1,0]]
输出：true
解释：总共有 2 门课程。学习课程 1 之前，你需要完成课程 0 。这是可能的。
```

**示例 2：**

```
输入：numCourses = 2, prerequisites = [[1,0],[0,1]]
输出：false
解释：总共有 2 门课程。学习课程 1 之前，你需要先完成课程 0 ；并且学习课程 0 之前，你还应先完成课程 1 。这是不可能的。
```



## 题解

遍历先修课程规则，对于每个课程保存当前先修课程数量（如果为0则代表没有先修课程），以及后续课程的具体课程号（因为修了该课程后，后续课程的先修课程数量都可以减一）

通过队列依次修课程直到没有课程可以修为止（入度为0的剩余课程）

最后看修的课程和总课程是否一致

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        if(numCourses <= 0) {
            return true;
        }
        int n = prerequisites.length;
        if(n == 0) {
            return true;
        }
        //每个课程先修课程数量(入度)
        int[] beforeClass = new int[numCourses];
        //每个课程的是哪些课程的前置课程
        Set<Integer>[] afterClass = new HashSet[numCourses];

        //初始化每个课程HashSet
        for(int i = 0; i < numCourses; i++) {
            afterClass[i] = new HashSet<>();
        }
        //
        for(int[] rule : prerequisites) {
            //规则中前者需要先修后者的课，前者入度+1
            beforeClass[rule[0]]++;
            //对于需要先修课程，加入先修后可以修的具体课程
            afterClass[rule[1]].add(rule[0]);
        }

        //将入度为0的课程放入队列中，代表该课程不需要其他的先修课程
        Queue<Integer> queue = new LinkedList<>();
        for(int i = 0; i < numCourses; i++) {
            if(beforeClass[i] == 0) {
                queue.add(i);
            }
        }


        int cnt = 0;
        while(!queue.isEmpty()) {
            //将入度为0的课程依次取出，代表该课程可以完成学习
            Integer target = queue.poll();
            cnt++;
            //遍历取出课程的后续课程，使它的入度减一
            for(int after : afterClass[target]) {
                beforeClass[after]--;
                //如果入度为0代表没有前修课程了，放入队列等待下次取出
                if(beforeClass[after] == 0) {
                    queue.add(after);
                }
            }
        }
        //如果所有的课程都被取出，曾入度都为0，代表课程可以被修完
        return cnt == numCourses;
    }
}
```

