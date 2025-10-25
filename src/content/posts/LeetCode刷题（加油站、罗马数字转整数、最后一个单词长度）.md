---
title: LeetCode刷题（加油站、罗马数字转整数、最后一个单词长度）
published: 2025-10-25
updated: 2025-10-25
description: '加油站、罗马数字转整数、最后一个单词长度'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 加油站

## 题目描述

在一条环路上有 `n` 个加油站，其中第 `i` 个加油站有汽油 `gas[i]` 升。

你有一辆油箱容量无限的的汽车，从第 `i` 个加油站开往第 `i+1` 个加油站需要消耗汽油 `cost[i]` 升。你从其中的一个加油站出发，开始时油箱为空。

给定两个整数数组 `gas` 和 `cost` ，如果你可以按顺序绕环路行驶一周，则返回出发时加油站的编号，否则返回 `-1` 。如果存在解，则 **保证** 它是 **唯一** 的。

**示例 1:**

```
输入: gas = [1,2,3,4,5], cost = [3,4,5,1,2]
输出: 3
解释:
从 3 号加油站(索引为 3 处)出发，可获得 4 升汽油。此时油箱有 = 0 + 4 = 4 升汽油
开往 4 号加油站，此时油箱有 4 - 1 + 5 = 8 升汽油
开往 0 号加油站，此时油箱有 8 - 2 + 1 = 7 升汽油
开往 1 号加油站，此时油箱有 7 - 3 + 2 = 6 升汽油
开往 2 号加油站，此时油箱有 6 - 4 + 3 = 5 升汽油
开往 3 号加油站，你需要消耗 5 升汽油，正好足够你返回到 3 号加油站。
因此，3 可为起始索引。
```

**示例 2:**

```
输入: gas = [2,3,4], cost = [3,4,3]
输出: -1
解释:
你不能从 0 号或 1 号加油站出发，因为没有足够的汽油可以让你行驶到下一个加油站。
我们从 2 号加油站出发，可以获得 4 升汽油。 此时油箱有 = 0 + 4 = 4 升汽油
开往 0 号加油站，此时油箱有 4 - 3 + 2 = 3 升汽油
开往 1 号加油站，此时油箱有 3 - 3 + 3 = 3 升汽油
你无法返回 2 号加油站，因为返程需要消耗 4 升汽油，但是你的油箱只有 3 升汽油。
因此，无论怎样，你都不可能绕环路行驶一周。
```



## 题解

解法一：暴力拆解（超时）

依次遍历数组元素作为起点，直到找到一个可以返回到起始位置的地方

```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int n = gas.length;
        for(int i = 0; i < n; i++) {
            int res = gas[i];
            for(int j = i; ; j = (j + 1) % n) {
                if(res < cost[j]) {
                    break;
                }
                if((j + 1) % n == i) {
                    return i;
                }
                res = res - cost[j] + gas[(j + 1) % n];
            } 
        }
        return -1;
    }
}
```



解法二：

贪心算法，维护total和current两个数，total代表总的汽油和需要消耗的汽油的差值，如果差值小于零则怎样都不可能开到原点。

而current代表从start开始剩余的油量，如果这个数小于零，则代表当前start是不能完成路程的，需要将start设置为下一个开始点，并将current重置

当total大于0的时候，我们的start开始积累的油量就一定可以覆盖0 - start-1 的消耗

```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int n = gas.length;
        int total = 0;
        int current = 0;
        int start = 0;
        for(int i = 0; i < n; i++) {
            total += gas[i] - cost[i];
            current += gas[i] - cost[i];
            if(current < 0) {
                start = i + 1;
                current = 0;
            }
        }
        return total >= 0 ? start : -1;
    }
}
```



# 罗马数字转整数

## 题目描述

罗马数字包含以下七种字符: `I`， `V`， `X`， `L`，`C`，`D` 和 `M`。

```
字符          数值
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```

例如， 罗马数字 `2` 写做 `II` ，即为两个并列的 1 。`12` 写做 `XII` ，即为 `X` + `II` 。 `27` 写做 `XXVII`, 即为 `XX` + `V` + `II` 。

通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 `IIII`，而是 `IV`。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 `IX`。这个特殊的规则只适用于以下六种情况：

- `I` 可以放在 `V` (5) 和 `X` (10) 的左边，来表示 4 和 9。
- `X` 可以放在 `L` (50) 和 `C` (100) 的左边，来表示 40 和 90。 
- `C` 可以放在 `D` (500) 和 `M` (1000) 的左边，来表示 400 和 900。

给定一个罗马数字，将其转换成整数。

**示例 1:**

```
输入: s = "III"
输出: 3
```

**示例 2:**

```
输入: s = "IV"
输出: 4
```

**示例 3:**

```
输入: s = "IX"
输出: 9
```

**示例 4:**

```
输入: s = "LVIII"
输出: 58
解释: L = 50, V= 5, III = 3.
```

**示例 5:**

```
输入: s = "MCMXCIV"
输出: 1994
解释: M = 1000, CM = 900, XC = 90, IV = 4.
```



## 题解

解法一：存储所有存在的可能，优先选取更大的组合

```java
class Solution {
    public int romanToInt(String s) {
        Map<String, Integer> map = new HashMap<>();
        map.put("I", 1);
        map.put("IV", 4);
        map.put("V", 5);
        map.put("IX", 9);
        map.put("X", 10);
        map.put("XL", 40);
        map.put("L", 50);
        map.put("XC", 90);
        map.put("C", 100);
        map.put("CD", 400);
        map.put("D", 500);
        map.put("CM", 900);
        map.put("M", 1000);
        int result = 0;
        for(int i = 0; i < s.length();) {
            if(i + 1 < s.length() && map.containsKey(s.substring(i, i + 2))) {
                result += map.get(s.substring(i, i + 2));
                i += 2;
            } else {
                result += map.get(s.substring(i, i + 1));
                i++;
            }
        }
        return result;
    }
}
```



解法二：

通常情况，假如左边的数字都比右边的大，则只需要将每个字符进行拆分相加即可，但是如果由更大的的数字再右侧，则这个数字应该被减去，而非相加

```java
class Solution {
    public static Map<Character, Integer> map = new HashMap<>();
    static {
        map.put('I', 1);
        map.put('V', 5);
        map.put('X', 10);
        map.put('L', 50);
        map.put('C', 100);
        map.put('D', 500);
        map.put('M', 1000);
    }
    public int romanToInt(String s) {
        int result = 0;
        int n = s.length();
        for(int i = 0; i < n; i++) {
            int num = map.get(s.charAt(i));
            if(i < n - 1 && num < map.get(s.charAt(i + 1))) {
                result -= num;
            } else {
                result += num;
            }
        }
        return result;
    }
}
```



# 最后一个单词长度

## 题目描述

给你一个字符串 `s`，由若干单词组成，单词前后用一些空格字符隔开。返回字符串中 **最后一个** 单词的长度。

**单词** 是指仅由字母组成、不包含任何空格字符的最大子字符串。

**示例 1：**

```
输入：s = "Hello World"
输出：5
解释：最后一个单词是“World”，长度为 5。
```

**示例 2：**

```
输入：s = "   fly me   to   the moon  "
输出：4
解释：最后一个单词是“moon”，长度为 4。
```

**示例 3：**

```
输入：s = "luffy is still joyboy"
输出：6
解释：最后一个单词是长度为 6 的“joyboy”。
```



## 题解

解法一：家等通知解法，用split拆分，取最后一个单词长度即可

```java
class Solution {
    public int lengthOfLastWord(String s) {
        String[] strs = s.split(" ");
        int n = strs.length;
        String target = strs[n - 1];
        return target.length();
    }
}
```

解法二：从后往前排除空格后计数直到再次遇到空格或者结束

```java
class Solution {
    public int lengthOfLastWord(String s) {
        char[] arr = s.toCharArray();
        int n = arr.length;
        int result = 0;
        int i = n - 1;
        while(i >= 0 && arr[i] == ' ') {
           i--;
        }
        while(i >= 0 && arr[i] != ' ') {
            result++;
            i--;
        }
        return result;
    }
}
```

