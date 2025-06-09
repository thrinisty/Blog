---
title: MyBatis笔记（入门使用）
published: 2025-06-10
updated: 2025-06-10
description: 'MyBatis的入门使用'
image: ''
tags: [MyBatis]
category: 'Frame'
draft: false 
---

# MyBatis笔记

接下来的几天开始MyBatis的学习，用到的课程式动力节点杜老师的B站网课

<iframe 
    width="100%" 
    height="468" 
    src="//player.bilibili.com/player.html?bvid=BV1JP4y1Z73S&p=1&autoplay=false" 
    scrolling="no" 
    border="0" 
    frameborder="no" 
    framespacing="0" 
    allowfullscreen="true">
</iframe>

本来打算用尚硅谷的课程，但是18年的视频对现在来说以及太老了，jar包之类的都是通过手动引入的，非常的不方便，而且老师的课程也算是细康，讲的挺深入的

话不多说开始今天的学习



## MyBatis入门配置

相对于JDBC，MyBatis对于JDBC进行了封装，可以避免JDBC的一些缺陷，例如sql语句写死，违反OCP开闭原则

另外对于?的赋值非常繁琐，通过反射机制可以轻松实现对象的复制，参数的传入

在使用的时候可以通过Maven进行MyBatis的导入

```html
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.13</version> <!-- 以最新版本为准 -->
    </dependency>
</dependencies>
```

