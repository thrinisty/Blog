---
title: Spring笔记（实例化方式，生命周期，循环依赖）
published: 2025-05-25
updated: 2025-05-25
description: 'Bean的多种实例化方式，十部生命周期，Bean的循环依赖'
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



## Bean生命周期

Bean的生命周期：Bean对象的创建，销毁是在什么时候进行的

### 五步生命周期

Bean实例化 -> Bean属性赋值 -> 初始化Bean -> 使用Bean -> 销毁Bean

```java
package com.thrinisty.bean;

public class User {
    private String name;

    public void setName(String name) {
        System.out.println("第二步：属性赋值");
        this.name = name;
    }

    public User() {
        System.out.println("第一步：User无参构造器");
    }

    public void initBean() {
        System.out.println("第三步：初始化Bean");
    }

    public void destroyBean() {
        System.out.println("第五步：销毁Bean");
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

Spring中指定初始，销毁方法

```java
<bean id="user" class="com.thrinisty.bean.User"
    init-method="initBean" destroy-method="destroyBean">
    <property name="name" value="Jack"/>
</bean>
```

测试程序

```java
@Test
public void test01() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    User user = context.getBean("user", User.class);
    System.out.println("第四步：使用Bean" + user);
    ClassPathXmlApplicationContext cac = (ClassPathXmlApplicationContext)context;
    cac.close();
}
```

注意要使用close方法销毁Context需要向下转型为ClassPathXmlApplicationContext

```
第一步：User无参构造器
第二步：属性赋值
第三步：初始化Bean
第四步：使用BeanUser{name='Jack'}
第五步：销毁Bean
```



### 七步生命周期

Bean实例化 -> Bean属性赋值 -> 初始化Bean -> 使用Bean -> 销毁Bean

在初始化Bean前后可以各加一个生命期，通过Bean后处理器实现

执行Bean后处理器before方法-> 初始化Bean -> 执行Bean后处理器after方法



创建Bean后处理器

```java
public class LogBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("Before BeanPostProcessor");
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("After BeanPostProcessor");
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}
```

Spring配置，对所有当前的xml中的Bean对象都有用

```html
<bean class="com.thrinisty.bean.LogBeanPostProcessor"/>
```

```
第一步：User无参构造器
第二步：属性赋值
Before BeanPostProcessor
第三步：初始化Bean
After BeanPostProcessor
第四步：使用BeanUser{name='Jack'}
第五步：销毁Bean
```



### 十步生命周期

1.在Bean后处理器Before方法之前：检测Bean是否实现BeanNameAware接口，BeanClassLoaderAware接口，BeanFactoryAware接口

```java
public class User implements BeanNameAware,
        BeanClassLoaderAware, BeanFactoryAware {
    private String name;

    @Override
    public void setBeanName(String name) {
        System.out.println("Bean名字");
    }

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println("Bean类加载器");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("BeanFactory地址");
    }
```

2.在Bean后处理器Before方法之后：检测Bean是否实现InitializingBean接口

```java
@Override
public void afterPropertiesSet() throws Exception {
    System.out.println("After PropertiesSet");
}
```

3.销毁Bean之前，使用Bean之后：检测Bean是否实现DisposableBean接口

```java
@Override
public void destroy() throws Exception {
    System.out.println("DisposableBean destroy");
}
```

```
第一步：User无参构造器
第二步：属性赋值

Bean名字
Bean类加载器
BeanFactory地址
Before BeanPostProcessor
After PropertiesSet

第三步：初始化Bean
After BeanPostProcessor
第四步：使用BeanUser{name='Jack'}
DisposableBean destroy

第五步：销毁Bean
```

:::warning

1. Spring只对singleton的Bean进行完整的生命周期管理
2. 如果是prototype作用域的Bean，Spring只负责将Bean初始化完毕，一旦用户获取到Bean，Spring就不再管理Bean生命周期（管理前八步）

:::



### Spring管理new对象

```java
@Test
public void test02(){
    Student student = new Student();
    //创建一个工厂对象，使用工厂对象完成new对象实例在Spring中的注册
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.registerSingleton("student", student);

    Student student1 = factory.getBean("student", Student.class);
    System.out.println(student);
    System.out.println(student1);
}
```

```
com.thrinisty.bean.Student@26b3fd41
com.thrinisty.bean.Student@26b3fd41
```

使用的是同一个对象



## Bean的循环依赖

循环依赖定义：A对象中有B属性，B对象中有A属性

创建夫妻两个对象类，注意避免循环调用toString

```java
package com.thrinisty.bean;

public class Husband {
    private String name;
    private Wife wife;

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Husband{" +
                "name='" + name + '\'' +
                ", wife=" + wife.getName() +
                '}';
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setWife(Wife wife) {
        this.wife = wife;
    }
}
```

```java
package com.thrinisty.bean;

public class Wife {
    private String name;
    private Husband husband;

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Wife{" +
                "name='" + name + '\'' +
                ", husband=" + husband.getName() +
                '}';
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setHusband(Husband husband) {
        this.husband = husband;
    }
}
```



### 单例模式

spring配置

```html
<bean id="husbandBean" class="com.thrinisty.bean.Husband">
    <property name="name" value="Jack"/>
    <property name="wife" ref="wifeBean"/>
</bean>
<bean id="wifeBean" class="com.thrinisty.bean.Wife">
    <property name="name" value="Rose"/>
    <property name="husband" ref="husbandBean"/>
</bean>
```

循环依赖(单例)在测试中没有问题

```java
@Test
public void test01() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    Husband husband = context.getBean("husbandBean", Husband.class);
    System.out.println(husband);
    Wife wife = context.getBean("wifeBean", Wife.class);
    System.out.println(wife);
}
```

singleton + setter ：Spring为Bean的管理主要分为两个阶段

1.Spring容器加载的时候，实例化Bean，只需要任意一个Bean实例化之后，马上进行“曝光”（不等属性赋值）

2.Bean曝光之后再进行属性赋值，调用set方法进行赋值

:::warning

只有scope使singleton情况下，Bean才会采取提前曝光

:::



### 多例模式

修改Bean的scope范围

```html
<bean id="husbandBean" class="com.thrinisty.bean.Husband" scope="prototype">
    <property name="name" value="Jack"/>
    <property name="wife" ref="wifeBean"/>
</bean>
<bean id="wifeBean" class="com.thrinisty.bean.Wife" scope="prototype">
    <property name="name" value="Rose"/>
    <property name="husband" ref="husbandBean"/>
</bean>
```

因为每一次赋值都会创建新的Bean对象，会出现异常

可以将husbandBean范围改为单例，就可以正常执行，不会出现循环创建Bean

:::tip

当两个循环依赖的Bean对象都是prototype才会发生循环创建异常

:::



### 构造注入

无法解决循环依赖

对于构造创建循环依赖的两个对象，是没有办法进行曝光赋值的，因为对象没有创建出来，创建对象依赖于另一个对象，无法执行set单例中的第一个阶段完成曝光
