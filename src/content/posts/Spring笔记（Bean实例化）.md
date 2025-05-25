---
title: Spring笔记（Bean实例化）
published: 2025-05-25
updated: 2025-05-25
description: 'Bean的多种实例化方式'
image: ''
tags: [Spring]
category: 'Frame'
draft: false 
---

# Spring笔记

​	之前的Spring的学习计划因为考试和作业被稍微耽搁了一下，原本计划是到30号以后再继续学习的，但是最近稍微忙了一下，把编译原理理论给复习完了（其实上课听了的话不会话太长时间）

​	考试在28号，打算在27号和28号的两天早上再做一做题目热手，24-26号这三天还是可以抽得出时间稍微学习一下。



## Bean实例化

:::note

Spring为Bean提供了多种实例化的方式，通常包括四种

1.通过构造方式创建实例化

2.通过简单工厂模式实例化

3.通过factory-bean实例化

4.通过FactoryBean接口实例化

:::



### 构造方法创建

通过静态方法创建Bean

```html
<bean id="personBean" class="com.thrinisty.bean.Person"/>
```

通过context利用配置文件中的Bean对象，自动调用无参构造器创建对象

```java
@Test
public void test01() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    Person person = context.getBean("personBean", Person.class);
    person.fun();
}
```



### 简单工厂模式实例化

通过factoryStar Bean对象，指定createStar方法创建Star实例

```html
<bean id="factoryStar" class="com.thrinisty.bean.StarFactory" factory-method="createStar"/>
```

```java
public class StarFactory {
    //简单工厂
    //静态工厂方法
    public static Star createStar() {
        return new Star();
    }
}
```

测试程序，传入工厂类，底层根据配置调用构造Star的方法

```java
@Test
public void test02() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    Star star = context.getBean("factoryStar", Star.class);
}
```



### FactoryBean实例化

工厂方法模式实例化

相比于简单工厂，通过实例方法构建对象

```java
public class GunFactory {
    //工厂模式方法，是实例方法
    public Gun createGun() {
        return new Gun();
    }
}
```

spring配置：先创建工厂bean对象，使用Bean对象的对应方法

分别通过factory-bean和factory-method指定

```html
<bean id="gunFactory" class="com.thrinisty.bean.GunFactory"/>
<bean id="getFactory" factory-bean="gunFactory" factory-method="createGun"/>
```

测试程序

```java
@Test
public void test03() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    Gun gun = context.getBean("getFactory", Gun.class);
}
```



### FactoryBean接口

当编写的Bean实现了FactoryBean接口，就可以不用指定factory-bean和factory-method，可以简化配置

实现FactoryBean接口的工厂类

```java
public class DogFactoryBean implements FactoryBean<Dog> {
    @Override
    public Dog getObject() throws Exception {
        return new Dog();
    }

    @Override
    public Class<?> getObjectType() {
        return Dog.class;
    }

    @Override
    public boolean isSingleton() {
        return FactoryBean.super.isSingleton();
    }
}
```

spring配置，相比于前一种方式不需要传入实例Bean对象以及对应方法

```html
<bean id="dogBean" class="com.thrinisty.bean.DogFactoryBean"/>
```

测试文件

```java
@Test
public void test04() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    Dog dog = context.getBean("dogBean", Dog.class);
    System.out.println(dog);
}
```

通过工厂Factory相比较于直接从全路径获取Class对象，可以进行加工操作，有一些自己的优势



:::tip

BeanFactory与FactoryBean的区别

1. BeanFactory： Spring IoC中的顶级对象，译为Bean工厂，负责创建Bean对象
2. FactoryBean：是一个Bean，是一个可以辅助Spring实例化Bean对象的一个Bean对象，而在Spring中Bean分为普通Bean与工厂Bean

:::



## FactoryBean实际使用

以前我们使用set注入实现日期的赋值

```java
public class Student {
    private Date birth;

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    @Override
    public String toString() {
        return "Student{" +
                "birth=" + birth +
                '}';
    }
}
```

在配置中关联nowTime Bean对象，实现日期的注入

```html
<bean id="student" class="com.thrinisty.bean.Student">
    <property name="birth" ref="nowTime"/>
</bean>
<bean id="nowTime" class="java.util.Date"/>
```

但是这样没有办法调用有参构造器传入指定的时间，不能作为生日使用（当前时间），接下来我们使用FactoryBean注入Date日期

工厂Bean，辅助加工Date属性

```java
public class DateFactoryBean implements FactoryBean<Date> {
    private String strDate;

    public void setStrDate(String strDate) {
        this.strDate = strDate;
    }

    @Override
    public Date getObject() throws Exception {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        Date date = sdf.parse(strDate);
        return date;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }
}
```

构造配置，使用工厂Bean提供的Date对象关联student的日期类

```html
<bean id="student" class="com.thrinisty.bean.Student">
    <property name="birth" ref="birth"/>
</bean>
<bean id="birth" class="com.thrinisty.bean.DateFactoryBean">
    <property name="strDate" value="1980-10-11"/>
</bean>
```
