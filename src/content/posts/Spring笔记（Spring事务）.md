---
title: Spring笔记（Spring事务）
published: 2025-06-08
updated: 2025-06-08
description: 'Spring事务'
image: ''
tags: [Spring]
category: 'Frame'
draft: false 
---

# Spring笔记

## 事务

一个业务流程中通常需要多条DML语句联合才可以完成，必须同时成功和失败，这要才能保证前后的正确性。

事务可以通过AOP插入完成（底层也是通过AOP插入的），而Spring对于事务二次封装使得开发者更加方便使用事务，提高开发效率

引入相关依赖

```html
<!-- Spring Boot Starter AOP (包含 Spring AOP 核心功能) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.18</version> <!-- 请使用最新稳定版 -->
</dependency>
<!-- Spring JDBC (包含 spring-jdbc 和 spring-tx) -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>6.1.0</version> <!-- 使用最新稳定版 -->
</dependency>
<!-- MySQL Connector/J -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.33</version> <!-- 使用最新稳定版 -->
</dependency>
```

在事务层的两条SQL语句之间模拟发生了异常，另外的一条语句不再执行，这个时候，账户的钱少了10000块，发生丢失

数据操作层

```java
@Repository("accountDao")
public class AccountDaoImpl implements AccountDao {
    @Resource(name = "jdbc")
    private JdbcTemplate jdbcTemplate;

    @Override
    public Account selectByActno(String actno) {
        String sql = "select actno, balance from t_act where actno = ?";
        Account account = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(Account.class), actno);
        return account;
    }

    @Override
    public int update(Account act) {
        String sql = "update t_act set balance = ? where actno = ?";
        int update = jdbcTemplate.update(sql, act.getBalance(), act.getUsername());
        return update;
    }
}
```

控制层处理代码

```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {
    @Resource(name = "accountDao")
    private AccountDao accountDao;

    @Override
    public void transfer(String from, String to, double balance) {
        Account fromAct = accountDao.selectByActno(from);
        if (fromAct.getBalance() < balance) {
            throw new RuntimeException("余额不足");
        }
        Account toAct = accountDao.selectByActno(to);
        fromAct.setUsername(from);//这里非常奇怪获取对象的时候没有将用户的名称成功的复制到Bean对象中，于是手动处理一下
        toAct.setUsername(to);
        fromAct.setBalance(fromAct.getBalance() - balance);
        toAct.setBalance(toAct.getBalance() + balance);
        int update = accountDao.update(fromAct);
        if(true) {
            throw new RuntimeException("中途发送异常");
        }
        update += accountDao.update(toAct);
        if (update != 2) {
            throw new RuntimeException("转账失败");
        }
    }
}
```

测试程序

```java
@Test
    public void test() {
        ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
        AccountService accountService = context.getBean("accountService", AccountService.class);
        try {
            accountService.transfer("user1", "user2", 10000);
            System.out.println("转账成功");
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
```



## 使用事务方式

基于AOP封装，Spring提供了事务API供程序员使用，我们可以利用它解决上述问题

编程式事务：通过编写代码的方式实现对于事务的管理

声明式事务：基于注解方式，基于XML配置方式

使用注解是最多的

PlatformTransactionManager是事务管理器的核心接口，在Spring6中有两个实现，DataSourceTransactionManager：支持MyBatis，JdbcTemplate等事务管理，而JtaTransactionManager支持分布式事务管理

Spring配置

```html
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    <context:component-scan base-package="com"/>
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring6"/>
        <property name="username" value="root"/>
        <property name="password" value="654321"/>
    </bean>

    <bean id="jdbc" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

<!--    开启事务注解-->
    <tx:annotation-driven transaction-manager="txManager"/>
</beans>
```



加上@Transactional注释

```java
@Override
    @Transactional
    public void transfer(String from, String to, double balance) {
        //开启事务
```

即可在发生异常的时候发生回滚，非常的方便易用，这个注解也可以作用在类上，对其所有的方法进行使用

顺带提一嘴，通过Spring可以整合JUnit5测试框架，就不用再用getBean获取对象了，直接自动装配即可

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration("classpath:spring.xml")
public class BeanTest {
    @Autowired
    private AccountService accountService;

    @Test
    public void test() {
//        ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
//        AccountService accountService = context.getBean("accountService", AccountService.class);
        try {
            accountService.transfer("user1", "user2", 10000);
            System.out.println("转账成功");
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```



## 事务的属性

@Transactional注释的本身有很多的属性

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Reflective
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

    String[] label() default {};

    Propagation propagation() default Propagation.REQUIRED;
    //事务传播行为

    Isolation isolation() default Isolation.DEFAULT;
    //事务隔离级别

    int timeout() default -1;
    //事务超时时间

    String timeoutString() default "";

    boolean readOnly() default false;
    //是否只读，true为只读

    Class<? extends Throwable>[] rollbackFor() default {};
	//可以设置发生何种异常时回滚
    
    String[] rollbackForClassName() default {};

    Class<? extends Throwable>[] noRollbackFor() default {};
	////可以设置发生何种异常时不回滚
    String[] noRollbackForClassName() default {};
}
```

以下我们来挨个说明



### 事务传播行为

```
Propagation propagation() default Propagation.REQUIRED;
//事务传播行为
```

当我们用一个被事务标注标记的方法调用里另一个被事务标注标记的方法，就应该考虑事务是如何传递的，是新开启一个事务还是合并到原有事务中，事务的传播行为在Spring中有如下的七种枚举类型

REQUIRED:支持当前事务，如果不存在就新建一个（默认行为），使用原先事务

SUPPORTS: 支持当前事务，如果当前没有事务，就以非事务方法执行，随和

MANDATORY:必须运行在一个事务中，如果没有正在发生的事务抛出异常，强制性

REQUIRES_NEW:开启一个新的事务，如果一个事务以及存在，将存在的事物挂起（不存在嵌套，之前的事务挂起）

NOT_SUPPORTED:以非事务方式运行，如果事务存在，挂起当前事务

NEVER:以非事务方式运行，如果事务存在，抛出异常

NESTED（内嵌）:如果当前正有一个事务在进行中，则该方法应当运行在一个嵌套式事务中。被嵌套的事务可以独立于外层事务进行提交或者回滚，如果外层事务不存在，行为与 REQUIRED 一致



**实际使用**

我们假设有一个场景，一个被事务注解修饰的方法，调用了另一个被事务注解修饰的方法，后一个注解指定为REQUIRED，那么被调用的数条执行语句就会被加入到第一个方法的事务中，若此时此刻被调用的方法抛出异常，调用方法也会受到影响从而回滚

而如果第二个方法事务传播行为注解为REQUIRES_NEW，那么就会开启一个新的事务，如果其发生异常，第二个方法的数条执行语句回滚，与此同时第一个方法中如果针对于第二个方法抛出异常予以捕获，则第一个方法不会回滚

```java
@Transactional(propagation = Propagation.REQUIRED)
public void withdraw() {
    System.out.println("该方法也有事务");   
}
```



### 事务的隔离级别

之前的MySQL中学过，现在我们复习一下

脏读：当一个事务读取另一个事务尚未提交的修改时，产生脏读

不可重复读：同一查询在同一事物中多次进行，由于其他提交事务所做的修改或者删除，每次返回结果不同的结果集，产生不可重复读

幻读：同意查询在同一事物中多次进行，由于其他提交事务的插入操作，每次返回不同的结果集，产生了幻读



#### 事务隔离表格

事务隔离级别：MySQL隔离级别定义了事物与事务之间的隔离程度

| 隔离级别 | 丢失修改 | 脏读     | 不可重复读 | 幻读     |
| :------: | -------- | -------- | ---------- | -------- |
| 读未提交 | 不会发生 | 可能发生 | 可能发生   | 可能发生 |
| 读已提交 | 不会发生 | 不会发生 | 可能发生   | 可能发生 |
| 可重复读 | 不会发生 | 不会发生 | 不会发生   | 可能发生 |
| 可串行化 | 不会发生 | 不会发生 | 不会发生   | 不会发生 |



#### Spring隔离级别

对应着上表的四种隔离级别，默认的隔离级别是根据所选数据库决定的

```java
public enum Isolation {
    DEFAULT(-1),
    READ_UNCOMMITTED(1),
    READ_COMMITTED(2),
    REPEATABLE_READ(4),
    SERIALIZABLE(8);

    private final int value;

    private Isolation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }
}
```

使用方式：

```java
@Transactional(isolation = Isolation.REPEATABLE_READ, propagation = Propagation.REQUIRED)
public void withdraw() {
    System.out.println("某一事务");
}
```



### 事务超时属性

事务的超时指的是，如果对应秒数流逝完毕的时候，而DML没有全部执行完毕，最终结果会选择回滚，在实际使用中通过timeout属性设置秒数

我们测试一下

```java
@Transactional(timeout = 10)
public void save(Account account) {
    try {
        accountDao.update(account);
        Thread.sleep(1000 * 20);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
}
```

在执行DML语句后睡眠20s，而超时时间是10s

但是从结果上显示，该DML语句任然被执行，这是因为在超时之前所有的DML都被执行完成

交换一下顺序，这样就会出现超时

```java
@Transactional(timeout = 5)
public void save(Account account) {
    try {
        Thread.sleep(1000 * 10);
        accountDao.update(account);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
}
```

这里是面试中常考的一点



### 只读事务

如果设置一个事务是只读事务，则该事务中不可以执行增删改语句，启用只读的时候，可以启动Spring的优化策略，提高select语句的执行效率

```java
@Transactional(readOnly = true)
public void getByActno(String actno) {
    Account account = accountDao.selectByActno(actno);
    account.setUsername(actno);
    System.out.println(account);
}
```

在实际使用为了增加效率可以启用一下



### 设置异常[不]回滚事务

在Spring事务中默认发生任何异常，进行回滚

我们也可以通过rollbackFor人为设置，表示发生对应异常或者其子类异常的时候才会发生回滚

```java
@Transactional(rollbackFor = Exception.class)
public void getByActno(String actno) {
    Account account = accountDao.selectByActno(actno);
    account.setUsername(actno);
    System.out.println(account);
}
```

也可以设置遇到对应异常不回滚

```java
@Transactional(noRollbackFor = NullPointerException.class)
public void save(Account account) {
    accountDao.update(account);
    throw new NullPointerException();
}
```

在测试程序中捕获到空指针异常，但是DML语句不发生回滚



## 事务全注解式开发

我们以上使用事物的时候使用到了spring的xml配置文件

```html
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    <context:component-scan base-package="com"/>
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring6"/>
        <property name="username" value="root"/>
        <property name="password" value="654321"/>
    </bean>

    <bean id="jdbc" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

<!--    开启事务注解-->
    <tx:annotation-driven transaction-manager="txManager"/>
</beans>
```

还是非常繁多的，我们一般在实际使用的时候，通过注解来进行配置

在配置类中用@Bean注解配置几个Bean对象

```java
@Configuration
@ComponentScan("com")
@EnableTransactionManagement//开启事务的注解
public class SpringConfig {
    @Bean(name = "dataSource")
    public DruidDataSource getDataSource() {
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring6");
        ds.setUsername("root");
        ds.setPassword("654321");
        return ds;
    }
    @Bean(name = "jdbc")
    public JdbcTemplate getJdbcTemplate() {
        return new JdbcTemplate(getDataSource());
    }
    @Bean(name = "txManager")
    public PlatformTransactionManager getTxManager() {
        return new DataSourceTransactionManager(getDataSource());
    }
}
```

测试程序

```java
@Test
public void test1() {
    ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
    IsolationService iso = context.getBean("i1", IsolationService.class);
    try {
        iso.getByActno("user1");
    } catch (Exception e) {
        System.out.println("捕获到异常");
    }
}
```

通过XML也可以实现，需要在Spring配置中用tx标签和aop标签指定需要配置的方法启用其事务，相比注解式开发确实有些复杂了，了解即可



# Spring总结

Spring的教程部分到此结束，其实还有一个用Spring配置MyBatis的部分，但是我还没有学到MyBatis，加下来的数天回进行MyBatis的学习，等到完成学习之后再会过头来Spring配置MyBatis部分的学习
