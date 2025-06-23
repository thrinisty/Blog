---
title: MyBatis笔记（javassist生成类）
published: 2025-06-23
updated: 2025-06-23
description: 'javassist生成类'
image: ''
tags: [MyBatis]
category: 'Frame'
draft: false 
---

# MyBatis笔记

前段时间终于将租的房子确定下来了，因为要熟悉一下周边环境，就稍微给自己放了几天假，在这几天把该买的，该收拾的都搞完了，今天是公司实习的第一天，希望在公司实习的同时不落下之前预定课业的学习



## javassist

### 基本使用

通过javassist我们只需要编写Dao接口，他可以帮我们动态的生成Dao实现代码

引入相关依赖

```html
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.29.2-GA</version> <!-- 使用最新版本 -->
</dependency>
```

```java
@Test
    public void test() {
        ClassPool pool = ClassPool.getDefault();
        CtClass ctClass = pool.makeClass("com.dao.DaoImpl");
        String methodCode = "public void insert() {System.out.println(123);}";
        try {
            CtMethod ctMethod = CtMethod.make(methodCode, ctClass);
            ctClass.addMethod(ctMethod);
            ctClass.toClass();

            //类加载
            Class<?> clz = Class.forName("com.dao.DaoImpl");
            Constructor<?> constructor = clz.getConstructor();
            Object object = constructor.newInstance();
            Method method = clz.getMethod("insert");
            method.invoke(object);

        } catch (Exception e) {
            System.out.println("发送错误");
        }
    }
```

注意这里在将类加载进入字节码的时候由于JDK17模块系统限制了反射访问某些关键类，会出现错误，我们通过添加JVM参数解决

```
--add-opens java.base/java.lang=ALL-UNNAMED
```



### 实现接口
