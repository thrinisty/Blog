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

# 注解式开发

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
- @Autowired： 通过类型自动装配
- @Qualifier：配合Autowired使用，在接口有不同实现类时候指定名字
- @Resource： jdk拓展，也是最推荐使用的复杂类型注入的方式

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



### Autowired、Qualifier注解

Autowired注解可以用来注入非简单类型，翻译为自动装配

单独使用Autowired注解，默认根据类型装配

如果需要根据名字进行装配，需要联合Qualifier注解使用

OrderDao接口

```java
public interface OrderDao {
    void insert();
}
```

OrderDaoImplForMySQL实现类

```java
@Repository("orderDaoImplForMySQL")
public class OrderDaoImplForMySQL implements OrderDao {
    @Override
    public void insert() {
        System.out.println("MySQL insert order");
    }
}
```

服务类，拥有实现类对象接口，使用@Autowired自动装配

```java
@Service("orderService")
public class OrderService {
    @Autowired
    private OrderDao orderDao;
    public void generate() {
        orderDao.insert();
    }
}
```

测试程序

```java
@Test
public void test() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    OrderService service = context.getBean("orderService", OrderService.class);
    service.generate();
}
```

```
MySQL insert order
```

:::tip

如果类只有一个构造方法，且属性和参数能对应上，可以省略@Autowired注解

:::



这种方式适用于一个接口对应的一个实现类，如果有两个实现类就不能完成自动装配，需要用Qualifier注解，实现按照名字装配

现有另外的一个NoSQL实现类

```java
@Repository("orderDaoImplForNoSQL")
public class OrderDaoImplForNoSQL implements OrderDao {
    @Override
    public void insert() {
        System.out.println("NoSQL Insert Order");
    }
}
```

注入的时候利用@Qualifier("orderDaoImplForNoSQL")指定注入的Bean对象

```java
@Service("orderService")
public class OrderService {
    @Autowired
    @Qualifier("orderDaoImplForNoSQL")
    private OrderDao orderDao;
    public void generate() {
        orderDao.insert();
    }
}
```



### Resource注解

Resource注解可以完成非简单类型的注入，是Java标准规范的一部分，是标准注解，而Autowired注解是Spring框架自己的

Resource注解默认根据名字进行装配，未指定名字时，使用属性名作为name，找不到会自动启动类型装配

使用：直接指定名字完成按照名字装配

```java
@Service("orderService")
public class OrderService {
    @Resource(name = "orderDaoImplForNoSQL")
    private OrderDao orderDao;
    public void generate() {
        orderDao.insert();
    }
}
```



## 全注解开发

可以通过一个类代替spring配置文件，以后不需要使用Spring配置文件

```java
package com.bean3;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("com")
public class SpringConfig {
}
```

测试程序

```java
@Test
public void testNoXML() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
    OrderService service = context.getBean("orderService", OrderService.class);
    service.generate();
}
```



# JdbcTemplate

是Spring提供的一个JDBC模板类，是对JDBC的封装，简化JDBC代码

除此之外，你还可以不使用它，而是用例如MyBatis等ORM框架，我还没有学过MyBatis，先从JdbcTemplate入门使用一下

## 基本使用

jdbc依赖以及mysql驱动

```html
<!-- Spring JDBC (包含 spring-jdbc 和 spring-tx) -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>6.0.9</version> <!-- 使用最新稳定版 -->
</dependency>
<!-- MySQL Connector/J -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.33</version> <!-- 使用最新稳定版 -->
</dependency>
```

实现一个DataSource的实现类，重写getConnection方法，并纳入Spring管理

```java
public class MyDateSource implements DataSource {
    private String driver;
    private String url;
    private String username;
    private String password;
    @Override
    public Connection getConnection() throws SQLException {
        try {
            Class.forName(driver);
            Connection connection = DriverManager.getConnection(url, username, password);
            return connection;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
```

Spring配置：配置了一个ds数据源Bean对象，注入各个参数，用内置的jdbcTemplate对象传入ds Bean对象到 dataSource参数

```html
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="ds" class="bean.MyDateSource">
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring6"/>
        <property name="username" value="root"/>
        <property name="password" value="654321"/>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="ds"/>
    </bean>
</beans>
```

测试程序：增加数据

```java
@Test
public void test() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = context.getBean("jdbcTemplate", JdbcTemplate.class);
    String sql = "insert into user (id, name, password) values (?, ?, ?)";
    int update = jdbcTemplate.update(sql, 3, "Lory", "135788");
    System.out.println(update);
}
```

```
1
```

测试程序：修改数据

```java
@Test
public void test() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = context.getBean("jdbcTemplate", JdbcTemplate.class);
    String sql = "update user set name=? where id=?";
    int update = jdbcTemplate.update(sql, "Joker", 1);
    System.out.println(update);
}
```

```
1
```

测试程序：删除数据

```java
@Test
public void test() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = context.getBean("jdbcTemplate", JdbcTemplate.class);
    String sql = "delete from user where id=?";
    int update = jdbcTemplate.update(sql, 3);
    System.out.println(update);
}
```

```
1
```

测试程序：查询单个数据

```java
@Test
public void test() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = context.getBean("jdbcTemplate", JdbcTemplate.class);
    String sql = "select * from user where id = ? ";
    User user = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(User.class), 1);
    System.out.println(user);
}
```

```
User{id=1, name='Joker', password='123456'}
```

测试程序：多个查询结果，返回List集合

```java
@Test
public void test() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = context.getBean("jdbcTemplate", JdbcTemplate.class);
    String sql = "select * from user";
    List<User> list = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(User.class));
    for(User user : list){
        System.out.println(user);
    }
}
```

```
User{id=1, name='Joker', password='123456'}
User{id=2, name='李四', password='654321'}
```

测试程序：查询单个值

```java
@Test
public void test() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = context.getBean("jdbcTemplate", JdbcTemplate.class);
    String sql = "select count(1) from user";
    Integer integer = jdbcTemplate.queryForObject(sql, int.class);
    System.out.println(integer);
}
```



## 整合Druid连接池

引入依赖

```html
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.18</version> <!-- 请使用最新稳定版 -->
</dependency>
```

Spring配置

```html
<bean id="ds" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/spring6"/>
    <property name="username" value="root"/>
    <property name="password" value="654321"/>
</bean>

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="ds"/>
</bean>
```

使用结果：看到输出了一些日志信息

```
21:28:46.235 [main] INFO com.alibaba.druid.pool.DruidDataSource -- {dataSource-1} inited
2
```

