---
title: LeetCode刷题（二叉树[反]序列化）
published: 2025-10-05
updated: 2025-10-05
description: '二叉树的序列化和反序列化'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 二叉树的序列化和反序列化

## 题目描述

序列化是将一个数据结构或者对象转换为连续的比特位的操作，进而可以将转换后的数据存储在一个文件或者内存中，同时也可以通过网络传输到另一个计算机环境，采取相反方式重构得到原数据。

请设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，你只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。

**提示:** 输入输出格式与 LeetCode 目前使用的方式一致，详情请参阅 [LeetCode 序列化二叉树的格式](https://support.leetcode.cn/hc/kb/article/1567641/)。你并非必须采取这种方式，你也可以采用其他的方法解决这个问题。

**示例 1：**

![239](../images/239.jpg)

```
输入：root = [1,2,3,null,null,4,5]
输出：[1,2,3,null,null,4,5]
```

**示例 2：**

```
输入：root = []
输出：[]
```

**示例 3：**

```
输入：root = [1]
输出：[1]
```

**示例 4：**

```
输入：root = [1,2]
输出：[1,2]
```



## 题解

二叉树的层序遍历思路，在遍历的过程中将节点中的数据添加到StringBuilder中，不一样的是当node不为空的时候左右节点就会被放到队列中在下一轮的循环中判断null，而不是原先的只有左右节点不为空的情况下才放入队列

在反序列化的时候按照 "," 进行拆分，同样的也设立一个队列，层序的添加节点的左右节点，如果当前数组索引的值不为null，则将其创建写入到队列中并且使其成为当前节点的左/右节点（按照顺序分别构造左右节点）

```java
public class Codec {

    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        if(root == null) {
            return "[]";
        }
        StringBuilder result = new StringBuilder("[");
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if(node != null) {
                result.append(node.val + ",");
                queue.offer(node.left);
                queue.offer(node.right);
            } else {
                result.append("null" + ",");
            }
        }
        result.deleteCharAt(result.length() - 1);
        result.append("]");
        return result.toString();
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        if(data.equals("[]")) {
            return null;
        }
        String[] vals = data.substring(1, data.length() - 1).split(",");
        TreeNode root = new TreeNode(Integer.parseInt(vals[0]));
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        int index = 1;
        while(!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if(!vals[index].equals("null")) {
                node.left = new TreeNode(Integer.parseInt(vals[index]));
                queue.offer(node.left);
            }
            index++;
            if(!vals[index].equals("null")) {
                node.right = new TreeNode(Integer.parseInt(vals[index]));
                queue.offer(node.right);
            }
            index++;
        }
        return root;
    }
}
```

