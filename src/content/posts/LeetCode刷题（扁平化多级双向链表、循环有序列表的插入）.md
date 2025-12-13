---
title: LeetCode刷题（扁平化多级双向链表、循环有序列表的插入）
published: 2025-12-07
updated: 2025-12-07
description: '扁平化多级双向链表、循环有序列表的插入'
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 扁平化多级双向链表

## 题目描述

多级双向链表中，除了指向下一个节点和前一个节点指针之外，它还有一个子链表指针，可能指向单独的双向链表。这些子列表也可能会有一个或多个自己的子项，依此类推，生成多级数据结构，如下面的示例所示。

给定位于列表第一级的头节点，请扁平化列表，即将这样的多级双向链表展平成普通的双向链表，使所有结点出现在单级双链表中。

 

**示例 1：**

```
输入：head = [1,2,3,4,5,6,null,null,null,7,8,9,10,null,null,11,12]
输出：[1,2,3,7,8,11,12,9,10,4,5,6]
解释：
```

输入的多级列表如下图所示：

![498](../images/498.png)

扁平化后的链表如下图：

![499](../images/499.png)

**示例 2：**

```
输入：head = [1,2,null,3]
输出：[1,3,2]
解释：

输入的多级列表如下图所示：

  1---2---NULL
  |
  3---NULL
```

**示例 3：**

```
输入：head = []
输出：[]
```



## 题解

依次遍历节点，记录每次遍历到节点的下一个节点，如果当前节点拥有子节点则将当前节点指向其子节点，并将后续的子链表加入到当前节点的后面，最后用子链表的最后一个元素指向上一级的下一个节点

```java
class Solution {
    public Node flatten(Node head) {
        Node cur = head;
        while(cur != null) {
            if(cur.child != null) {
                Node next = cur.next;
                cur.next = cur.child;
                cur.child.prev = cur;
                cur.child = null;
                Node last = cur.next;
                while(last.next != null) {
                    last = last.next;
                }
                if(next != null) {
                    last.next = next;
                    next.prev = last;
                }
            }
            cur = cur.next;
        }
        return head;
    }
}
```





# 循环有序列表的插入

## 题目描述

给定**循环单调非递减列表**中的一个点，写一个函数向这个列表中插入一个新元素 `insertVal` ，使这个列表仍然是循环升序的。

给定的可以是这个列表中任意一个顶点的指针，并不一定是这个列表中最小元素的指针。

如果有多个满足条件的插入位置，可以选择任意一个位置插入新的值，插入后整个列表仍然保持有序。

如果列表为空（给定的节点是 `null`），需要创建一个循环有序列表并返回这个节点。否则。请返回原先给定的节点。

 

**示例 1：**

![500](../images/500.jpg)


```
输入：head = [3,4,1], insertVal = 2
输出：[3,4,1,2]
解释：在上图中，有一个包含三个元素的循环有序列表，你获得值为 3 的节点的指针，我们需要向表中插入元素 2 。新插入的节点应该在 1 和 3 之间，插入之后，整个列表如上图所示，最后返回节点 3 。
```

![501](../images/501.jpg)

**示例 2：**

```
输入：head = [], insertVal = 1
输出：[1]
解释：列表为空（给定的节点是 null），创建一个循环有序列表并返回这个节点。
```

**示例 3：**

```
输入：head = [1], insertVal = 0
输出：[1,0]
```



## 题解

先遍历一次找出链表中的最大最小节点，之后判断插入的值是否在最小-最大范围内，如果不是则将其插入到最大之后的位置，反之，则遍历最小-最大范围内合适的位置插入

```java
class Solution {
    public Node insert(Node head, int insertVal) {
        Node node = new Node(insertVal);
        node.next = node;
        if(head == null) {
            return node;
        }
        Node p = head.next;
        Node max = head;
        while(p != head) {
            if(p.val >= max.val) {
                max = p;
            }
            p = p.next;
        }

        Node min = max.next;
        if(insertVal < min.val || insertVal > max.val) {
            node.next = max.next;
            max.next = node;
        } else {
            p = min;
            while(true) {
                if(p.val <= insertVal && p.next.val >= insertVal) {
                    break;
                }
                p = p.next;
            }
            node.next = p.next;
            p.next = node;
        }
        return head;
    }
}
```

