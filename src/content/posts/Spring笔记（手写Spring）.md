---
title: Spring笔记（手写Spring）
published: 2025-05-26
updated: 2025-05-26
description: '简单地手写一个Spring框架'
image: ''
tags: [Spring]
category: 'Frame'
draft: false 
---

# Spring笔记

## 回顾反射

写一个类，通过反射执行方法

```java
package com.thrinisty.bean;

public class User {
    private String name;
    private int age;

    public User() {}

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

反射的调用步骤

```java
public class Main {
    public static void main(String[] args) throws Exception{
        Class<?> cls = Class.forName("com.thrinisty.bean.User");
        Constructor<?> constructor = cls.getConstructor();
        Object user = constructor.newInstance();

        Field nameField = cls.getDeclaredField("name");
        Field ageField = cls.getDeclaredField("age");

        Method setName = cls.getMethod("setName", nameField.getType());
        setName.invoke(user, "John");
        Method setAge = cls.getMethod("setAge", ageField.getType());
        setAge.invoke(user, 18);
        System.out.println(user);
    }
}
```



## 手写Spring
