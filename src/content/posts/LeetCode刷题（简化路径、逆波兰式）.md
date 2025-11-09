---
title: LeetCode刷题（简化路径、逆波兰式）
published: 2025-11-06
updated: 2025-11-06
description: ''
image: ''
tags: [LeetCode]
category: 'LeetCode'
draft: false 
---

# 简化路径

## 题目描述

给你一个字符串 `path` ，表示指向某一文件或目录的 Unix 风格 **绝对路径** （以 `'/'` 开头），请你将其转化为 **更加简洁的规范路径**。

在 Unix 风格的文件系统中规则如下：

- 一个点 `'.'` 表示当前目录本身。
- 此外，两个点 `'..'` 表示将目录切换到上一级（指向父目录）。
- 任意多个连续的斜杠（即，`'//'` 或 `'///'`）都被视为单个斜杠 `'/'`。
- 任何其他格式的点（例如，`'...'` 或 `'....'`）均被视为有效的文件/目录名称。

返回的 **简化路径** 必须遵循下述格式：

- 始终以斜杠 `'/'` 开头。
- 两个目录名之间必须只有一个斜杠 `'/'` 。
- 最后一个目录名（如果存在）**不能** 以 `'/'` 结尾。
- 此外，路径仅包含从根目录到目标文件或目录的路径上的目录（即，不含 `'.'` 或 `'..'`）。

返回简化后得到的 **规范路径** 。

 

**示例 1：**

**输入：**path = "/home/"

**输出：**"/home"

**解释：**

应删除尾随斜杠。

**示例 2：**

**输入：**path = "/home//foo/"

**输出：**"/home/foo"

**解释：**

多个连续的斜杠被单个斜杠替换。

**示例 3：**

**输入：**path = "/home/user/Documents/../Pictures"

**输出：**"/home/user/Pictures"

**解释：**

两个点 `".."` 表示上一级目录（父目录）。

**示例 4：**

**输入：**path = "/../"

**输出：**"/"

**解释：**

不可能从根目录上升一级目录。

**示例 5：**

**输入：**path = "/.../a/../b/c/../d/./"

**输出：**"/.../b/d"

**解释：**

`"..."` 在这个问题中是一个合法的目录名。



## 题解

先使用split进行拆分，对于..的时候进行弹栈，对于一般情况下将字符放入栈，构造StringBuilder依次将弹栈的元素插入开头，并在开头写上/

```java
class Solution {
    public String simplifyPath(String path) {
        String[] arr = path.split("/");
        Stack<String> stack = new Stack<>();
        for(String str : arr) {
            if(str.equals("..") && !stack.isEmpty()) {
                stack.pop();
            } 
            if(!str.equals(".") && !str.equals("") && !str.equals("..")) {
                stack.push(str);
            }
        }
        StringBuilder sb = new StringBuilder();
        while(!stack.isEmpty()) {
            sb.insert(0, stack.pop());
            sb.insert(0, "/");
        }
        if(sb.length() == 0) sb.append("/");
        return sb.toString();
    }
}
```





# 逆波兰式

## 题目描述

给你一个字符串数组 `tokens` ，表示一个根据 [逆波兰表示法](https://baike.baidu.com/item/逆波兰式/128437) 表示的算术表达式。

请你计算该表达式。返回一个表示表达式值的整数。

**注意：**

- 有效的算符为 `'+'`、`'-'`、`'*'` 和 `'/'` 。
- 每个操作数（运算对象）都可以是一个整数或者另一个表达式。
- 两个整数之间的除法总是 **向零截断** 。
- 表达式中不含除零运算。
- 输入是一个根据逆波兰表示法表示的算术表达式。
- 答案及所有中间计算结果可以用 **32 位** 整数表示。

**示例 1：**

```
输入：tokens = ["2","1","+","3","*"]
输出：9
解释：该算式转化为常见的中缀算术表达式为：((2 + 1) * 3) = 9
```

**示例 2：**

```
输入：tokens = ["4","13","5","/","+"]
输出：6
解释：该算式转化为常见的中缀算术表达式为：(4 + (13 / 5)) = 6
```

**示例 3：**

```
输入：tokens = ["10","6","9","3","+","-11","*","/","*","17","+","5","+"]
输出：22
解释：该算式转化为常见的中缀算术表达式为：
  ((10 * (6 / ((9 + 3) * -11))) + 17) + 5
= ((10 * (6 / (12 * -11))) + 17) + 5
= ((10 * (6 / -132)) + 17) + 5
= ((10 * 0) + 17) + 5
= (0 + 17) + 5
= 17 + 5
= 22
```



## 题解

构造一个栈用于存储数字，当发现符号的时候去除其中的两个数字做对应的计算，最后放入栈中，遍历结束之后从栈中取出结果即可

```java
class Solution {
    public int evalRPN(String[] tokens) {
        Stack<String> stack = new Stack<>();
        for(String token : tokens) {
            if(token.equals("*") || token.equals("/") || token.equals("+") || token.equals("-")) {
                String n2 = stack.pop();
                String n1 = stack.pop();
                int num1 = Integer.parseInt(n1);
                int num2 = Integer.parseInt(n2);
                int target = 0;
                switch(token) {
                    case "*" :
                        target = num1 * num2;
                    break;
                    case "/" :
                        target = num1 / num2;
                    break;
                    case "+" :
                        target = num1 + num2;
                    break;
                    case "-" :
                        target = num1 - num2;
                    break;
                }
                stack.push(target + "");
            } else {
                stack.push(token);
            }
        }
        return Integer.parseInt(stack.pop());
    }
}
```

