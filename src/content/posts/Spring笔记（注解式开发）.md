---
title: Spring笔记（注解式开发，JdbcTemplate）
published: 2025-05-31
updated: 2025-05-31
description: 'Spring IoC注解式开发，JdbcTemplate的使用'
image: ''
tags: [Spring]
category: 'Frame'
draft: false 
---

# Spring笔记

注解的存在是为了简化XML的配置，Spring 6倡导全注解式开发



## 注解细节回顾

```java
//元注解，标注注解的注解，表示可以出现在类上和字段上
@Target(value = {ElementType.TYPE, ElementType.FIELD})
//@Target({ElementType.TYPE, ElementType.FIELD})
//注解名为value，注解可以省略
//@Target(ElementType.TYPE)
//当数组中只有一个元素，大括号可以省略
@Retention(RetentionPolicy.RUNTIME)
//保持型策略，RUNTIME使注解保存在class文件中，被反射读取
public @interface Component {
    //定义注解属性
    String value();
    String name();
}
```

用注解标注User类

```java
@Component(value = "123", name = "jack")
public class User {
}
```

通过反射读取注解元素

```java
public class TestBean {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = Class.forName("com.thrinisty.bean.User");
        if(clazz.isAnnotationPresent(Component.class)) {
            //获取类上的注解
            Component annotation = clazz.getAnnotation(Component.class);
            System.out.println(annotation.value());
            System.out.println(annotation.name());
        }
    }
}
```



## 注解声明原理

现在有一个包名，扫描包下的所有类，当类上有@Component注解的时候，实例化对象，然后放入Map集合中

```java
public class TestBean {
    public static void main(String[] args) throws Exception {
        Map<String, Object> map = new HashMap<String, Object>();
        //包名
        String packageName = "com.thrinisty.bean";
        //获取路径
        String packagePath = packageName.replaceAll("\\.", "/");
        URL url = ClassLoader.getSystemClassLoader().getResource(packagePath);
        String path = url.getPath();//获取绝对路径
        File file = new File(path);
        File[] files = file.listFiles();
        for (File f : files) {
            String[] split = f.getName().split("\\.");
            String className = packageName + "." + split[0];
            //获取到包下的每个类全路径
            Class<?> clazz = Class.forName(className);
            if (clazz.isAnnotationPresent(Component.class)) {
                //获取注解
                Component annotation = clazz.getAnnotation(Component.class);
                String id = annotation.value();
                Constructor<?> constructor = clazz.getConstructor();
                Object object = constructor.newInstance();
                map.put(id, object);
            }
        }
        System.out.println(map.toString());
    }
}
```

```java
@Component("userBean")
public class User {
}
```

```java
@Component("orderBean")
public class Other {
}
```

```java
public class Vip {
}
```

被注解Component标注的会被反射实例化，并且放入map集合

```
{
orderBean=com.thrinisty.bean.Other@2e0fa5d3, userBean=com.thrinisty.bean.User@5010be6
}
```



## 声明Bean注解

:::tip

负责声明Bean的注解，常见的包括四个

- @Component：组件
- @Controller：控制器
- @Service：业务
- @Repository：仓库

:::

```java
package org.springframework.stereotype;
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
    String value() default "";
}
```

后面三个通过@AliasFor起一个别名，其实本实质上是组件，用哪一个都可以

```java
package org.springframework.stereotype;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```

为了增强代码的可读性：表示层建议用Controller，业务层建议用Service，数组操作层建议用Repository



### 使用方式

1.加入aop依赖

2.再Spring配置中添加context命名空间，并指定扫描的包

3.在Bean类上使用注解

```html
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
<!--        给Spring指定要扫描那些包-->
    <context:component-scan base-package="com.bean"/>
</beans>
```

注解修饰类

```java
@Component("userBean")
public class User {
}
```

如果不给value赋值，默认以类的首字母小写命名（vip）

```java
@Controller
public class Vip {
}
```

测试程序，和之前一样

```java
public class TestBean {
    @Test
    public void test() {
        ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
        Vip vip = context.getBean("vip", Vip.class);
        User user = context.getBean("userBean", User.class);
        System.out.println(user.toString() + " " + vip.toString());
    }
}
```



### 多包扫描

```java
package com.dao;

import org.springframework.stereotype.Component;

@Component
public class Manager {
}
```



第一种解决方式：用，隔开即可

```html
<context:component-scan base-package="com.bean, com.dao"/>
```

第二种解决方式：指定父包

```html
<context:component-scan base-package="com"/>
```



### 选择性实例化

假设现在又很多Bean，被四种注解标注，现在需要实例化其中一种注解的Bean，而其余的不参与实例化

#### 第一种解法

利用 use-default-filters="false 使所有Bean注解失效

```html
<context:component-scan base-package="com.bean" use-default-filters="false"/>
```

再使用<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>指定注解全路径，使其生效

```html
<context:component-scan base-package="com.bean" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```



#### 第二种解法

先全部生效，再关闭不需要的注解

```html
<context:component-scan base-package="com.bean" use-default-filters="true">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

```
Component Manager
Component User
```



测试程序

```java
public class TestBean {
    @Test
    public void test() {
        ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    }
}
```

```
Controller Servlet
Controller Vip
```



## 注入Bean注解

:::tip

负责注入Bean的注解，常见的包括四个

- @Value：专门用于注入简单类型（可以使用在属性和方法上）
- @Autowired
- @Qualifier
- @Resource

:::

### Value注解

用于注入简单类型

```java
@Component
public class MyDataSource implements DataSource {
    @Value("com.mysql,cj.jdbc.Driver")
    private String driver;
    @Value("jdbc:mysql://localhost:3306/spring6")
    private String url;
    @Value("root")
    private String username;
    @Value("123456")
    private String password;
    @Override
    public String toString() {
        return "MyDataSource{" +
                "driver='" + driver + '\'' +
                ", url='" + url + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
    ...
}
```

通过构造方法参数赋值完成注入

```java
@Component
public class MyDataSource implements DataSource {
    private String driver;
    private String url;
    private String username;
    private String password;

    public MyDataSource(@Value("com.mysql,cj.jdbc.Driver") String driver,
                        @Value("jdbc:mysql://localhost:3306/spring6")String url,
                        @Value("root")String username,
                        @Value("123456")String password) {
        this.driver = driver;
        this.url = url;
        this.username = username;
        this.password = password;
    }
```

测试程序

```java
@Test
public void test() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    MyDataSource dataSource = context.getBean("myDataSource", MyDataSource.class);
    System.out.println(dataSource);
}
```

```
MyDataSource{driver='com.mysql,cj.jdbc.Driver', url='jdbc:mysql://localhost:3306/spring6', username='root', password='123456'}
```



### Autowired注解

Autowired注解可以用来注入非简单类型，翻译为自动装配

单独使用注解，默认根据类型装配
