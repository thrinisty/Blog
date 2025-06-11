---
title: MyBatis笔记（CRUD）
published: 2025-06-11
updated: 2025-06-11
description: 'MyBatis增删改查'
image: ''
tags: [MyBatis]
category: 'Frame'
draft: false 
---

# MyBatis笔记

昨天编写了一个MyBatis的入门程序，成功的让他跑了起来，今天使用MyBatis更加深入的部分



## MyBatis工具类

我们可以封装一个工具类，便于我们获取到Session对象

```java
package com.batis.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;

public class SqlSessionUtil {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            sqlSessionFactory = new SqlSessionFactoryBuilder()
                    .build(Resources.getResourceAsStream("mybatis-config.xml"));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    
    private SqlSessionUtil() {}
    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```

测试程序，可以看到代码简洁许多

```java
@Test
public void testUtil() {
    SqlSession sqlSession = SqlSessionUtil.getSqlSession();
    int update = sqlSession.delete("deleteValue");
    System.out.println(update);
    sqlSession.commit();
    sqlSession.close();
}
```
