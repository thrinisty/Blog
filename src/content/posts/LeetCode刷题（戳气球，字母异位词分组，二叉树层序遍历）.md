---
title: LeetCode刷题（戳气球，字母异位词分组，层序遍历）
published: 2025-05-28
updated: 2025-05-28
description: '戳气球，字母异位词分组，层序遍历二叉树'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 戳气球

## 题目描述

有 `n` 个气球，编号为`0` 到 `n - 1`，每个气球上都标有一个数字，这些数字存在数组 `nums` 中。

现在要求你戳破所有的气球。戳破第 `i` 个气球，你可以获得 `nums[i - 1] * nums[i] * nums[i + 1]` 枚硬币。 这里的 `i - 1` 和 `i + 1` 代表和 `i` 相邻的两个气球的序号。如果 `i - 1`或 `i + 1` 超出了数组的边界，那么就当它是一个数字为 `1` 的气球。

求所能获得硬币的最大数量。

**示例 1：**

```
输入：nums = [3,1,5,8]
输出：167
解释：
nums = [3,1,5,8] --> [3,5,8] --> [3,8] --> [8] --> []
coins =  3*1*5    +   3*5*8   +  1*3*8  + 1*8*1 = 167
```

**示例 2：**

```
输入：nums = [1,5]
输出：10
```



## 题解

以现在的水平而言，不看题解只能够想到构造一个新的数组用于存储开始和结尾的1，看了题解后明确思路，通过三重循环进行二维数组的动态规划得出答案

:::tip

三重循环分别代表的意义：

1. 取对应区间的长度，从最小区间3开始
2. 取对应长度区间的起始位置，根据长度的增加，上界依次减小
3. 在长度区间内，遍历开区间的节点，迭代最大的区间结果最大值

:::

在起始len为3的时候，left和right都是0，其实也就是只有一个气球可以戳

在后续的len为4的时候会使用到len为3时存储的开区间分别给左右两侧赋值，即左右两侧的结果都是最大的情况，再根据剩下的节点左右乘以len长度对应的边界节点数字，相加就是len为4在最后戳对应结点的时候的最大结果，而res是为了记录遍历所有对应节点中剩余最后节点取置最大的结果，并将其写入结果集二维数组中，用以下一次个长度的使用

另外注意：因为是最后一个被戳爆的，所以它周边没有球了！没有球了！只有这个开区间首尾的 i 和 j 了！！
这就是为什么DP的状态转移方程是只和 i 和 j 位置的数字有关

```java
class Solution {
    public int maxCoins(int[] nums) {
        int n = nums.length;
        int temp[] = new int[n + 2];
        temp[0] = 1;
        temp[n + 1] = 1;
        for(int i = 0; i < n; i++) {
            temp[i + 1] = nums[i];
        } 
        int dp[][] = new int[n + 2][n + 2];
        for(int len = 3; len <= n + 2; len++) {
            for(int i = 0; i <= n + 2 - len; i++) {
                int res = 0;
                int r_b = i + len - 1;
                for(int k = i + 1; k < r_b; k++) {
                    int left = dp[i][k];
                    int right = dp[k][r_b];
                    res = Math.max(res, temp[i] * temp[k] * temp[r_b] + left + right);
                }
                dp[i][r_b] = res;
            }
        }
        return dp[0][n + 1];
    }
}
```



# 字母异位词分组

## 题目描述

给你一个字符串数组，请你将 **字母异位词** 组合在一起。可以按任意顺序返回结果列表。

**字母异位词** 是由重新排列源单词的所有字母得到的一个新单词。

**示例 1:**

```
输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出: [["bat"],["nat","tan"],["ate","eat","tea"]]
```

**示例 2:**

```
输入: strs = [""]
输出: [[""]]
```

**示例 3:**

```
输入: strs = ["a"]
输出: [["a"]]
```



## 题解

利用HashMap的key值不能重复的特性，依次遍历字符串数组元素，将遍历的当前字符串用char数组存储，利用数组排序后再次构造字符串，这样得到的异构字符串会被更改成为相同的字符串

从结果map集合中取出对应的List异构存储集合，利用getOrDefault对于不存在的key值创建List集合，将当前遍历的字符串str放入对应的List异构存储集合，最后再将这个集合放回map中进行更新

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap();
        for(String str : strs) {
            char[] arr = str.toCharArray();
            Arrays.sort(arr);
            String key = new String(arr);
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        List<List<String>> result = new ArrayList<>(map.values());
        return result;
    }
}
```



# 二叉树层序遍历

## 题目描述

给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

 

**示例 1：**

![169](../images/169.jpg)

```
输入：root = [3,9,20,null,null,15,7]
输出：[[3],[9,20],[15,7]]
```

**示例 2：**

```
输入：root = [1]
输出：[[1]]
```

**示例 3：**

```
输入：root = []
输出：[]
```



## 题解

通过一个队列的集合来进行层序遍历，相较于普通的层序遍历不区分层级，通过在遍历每一层的时候使用一个n变量计数，完成时对应层级结束，再取下一层计数n

:::tip

这里我犯了一个小错，没有用new创建新的list集合加入result，导致了每一层的结果都是15，7，之前也用过这种方式，需要注意一下

:::

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        List<Integer> list = new ArrayList<>();
        if(root == null) {
            return result;
        }
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root);
        while(!queue.isEmpty()) {
            list.clear();
            int n = queue.size();
            for(int i = 0; i < n; i++) {
                TreeNode node = queue.poll();
                list.add(node.val);
                if(node.left != null) {
                    queue.offer(node.left);
                }
                if(node.right != null) {
                    queue.offer(node.right);
                }
            }
            result.add(new ArrayList<Integer>(list));
        }
        return result;
    }
}
```

