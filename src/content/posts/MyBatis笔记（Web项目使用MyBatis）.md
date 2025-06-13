---
title: MyBatis笔记（Web项目使用MyBatis）
published: 2025-06-13
updated: 2025-06-13
description: 'Web项目使用MyBatis'
image: ''
tags: [MyBatis]
category: 'Frame'
draft: false 
---

# MyBatis笔记

今天我们用web使用以下MyBatis，顺便复习一下JavaWeb

用Maven Archetype创建一个Web工程

手动更新以下web.xml

```html
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0"
         metadata-complete="false">
</web-app>
```

:::warning

注意这里 metadata-complete="true"不支持注解式Servlet，需要改为false

:::



添加一些Maven依赖

```html
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.thrinisty</groupId>
  <artifactId>mybatis-003</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>mybatis-003 Maven Webapp</name>
  <url>http://maven.apache.org</url>

  <dependencies>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.33</version> <!-- 可替换为最新版本 -->
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.13</version> <!-- 以最新版本为准 -->
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.5.13</version>
    </dependency>
  </dependencies>
  <build>
    <finalName>mybatis-003</finalName>
  </build>
</project>
```

bean对象

```java
public class Account {
    private int id;
    private String actno;
    private double balance;
    ...
}
```



## 前端页面

前端转账页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>银行账户转账</title>
</head>
<body>
<form action="/bank/transfer" method="post">
    转出账号：<input type="text" name="fromActno"><br/>
    转出账号：<input type="text" name="toActno"><br/>
    转出金额：<input type="text" name="money"><br/>
    <input type="submit" value="转账">
</form>
</body>
</html>
```



## Servlet程序

Servlet程序，运用web.xml进行配置

```html
<servlet>
    <servlet-name>test</servlet-name>
    <servlet-class>com.bank.web.AccountServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>test</servlet-name>
    <url-pattern>/transfer</url-pattern>
</servlet-mapping>
```

```java
public class AccountServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String fromAccount = req.getParameter("fromActno");
        String toAccount = req.getParameter("toActno");
        double amount = Double.parseDouble(req.getParameter("money"));

        //创建服务层转账服务
        AccountService service = new AccountServiceImpl();
        try {
            service.transter(fromAccount, toAccount, amount);
            //调用视图完成展示结果
            resp.sendRedirect(req.getContextPath() + "/success.html");
        } catch (MoneyNotEnoughException e) {
            resp.sendRedirect(req.getContextPath() + "/error1.html");
            System.out.println("余额不足");
        } catch (TransferException e) {
            resp.sendRedirect(req.getContextPath() + "/error2.html");
            System.out.println("转账失败");
        }
    }
}
```



## 服务层

AccountService类，其中用到了AccountDaoImpl的数据操作对象实现类

```java
public class AccountServiceImpl implements AccountService {
    @Override
    public void transter(String fromActno, String toActno, double money) throws MoneyNotEnoughException {
        AccountDao accountDao = new AccountDaoImpl();
        //判断转出账户余额是否充足（select）
        Account fromAccount = accountDao.selectAccount(fromActno);
        if (fromAccount != null && money > fromAccount.getBalance()) {
            throw new MoneyNotEnoughException("余额不足");
        }
        //判断转出余额不足，提示用户
        //账户余额充足，更新转出账户（update）
        Account toAccount = accountDao.selectAccount(toActno);
        fromAccount.setBalance(fromAccount.getBalance() - money);
        toAccount.setBalance(toAccount.getBalance() + money);

        //更新转出账户余额（update）
        boolean flag1 = accountDao.updateAccount(fromAccount);
        boolean flag2 = accountDao.updateAccount(toAccount);
        if(!flag1 || !flag2){
            throw new TransferException("转账异常");
        }
    }
}
```



## Dao

### MyBatis相关配置

mybatis-config.xml

```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--    <settings>-->
<!--        <setting name="logImpl" value="STDOUT_LOGGING"/>-->
<!--    </settings>-->
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="654321"/>
            </dataSource>
        </environment>

    </environments>
    <mappers>
<!--        用于编写SQL语句的文件-->
        <mapper resource="AccountMapper.xml"/>
    </mappers>
</configuration>
```

AccountMapper.xml

```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="AccountMapper">
    <select id="selectAccount" resultType="com.bank.bean.Account">
        select * from t_act where actno = #{actno};
    </select>
    <update id="updateAccount">
        update t_act set balance = #{balance} where actno = #{actno};
    </update>
</mapper>
```



### Dao使用

Account数据操作对象

```java
public class AccountDaoImpl implements AccountDao {
    @Override
    public Account selectAccount(String actno) {
        SqlSession sqlSession = SqlSessionUtil.getSqlSession();
        Account account = (Account) sqlSession.selectOne("AccountMapper.selectAccount", actno);
        sqlSession.close();
        return account;
    }

    @Override
    public boolean updateAccount(Account act) {
        SqlSession sqlSession = SqlSessionUtil.getSqlSession();
        int update = sqlSession.update("AccountMapper.updateAccount", act);
        sqlSession.commit();
        sqlSession.close();
        return update == 1;
    }
}
```



## Utils工具类

用到的工具类，目的是获取到SqlSession对象

```java
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



## 事务处理

我们在上述处理中，如果在更新两个账户信息的中间发送异常，不会发生回滚，转出账户中的钱财消失，而转入账户的金额不变

我看教程中使用TreadLocal解决的，大体思路是运用线程使得操作共享一个SqlSession对象

但我没有学过TreadLocal，我的解决方法是重写写了一个方法，在方法中更新两个账户，并记录成功与否，如果其中有一个没有成功进行rollback回滚，否则提交并返回true

```java
@Override
public boolean updateTowAccount(Account act1, Account act2) {
    SqlSession sqlSession = SqlSessionUtil.getSqlSession();
    int update1 = sqlSession.update("AccountMapper.updateAccount", act1);
    int update2 = sqlSession.update("AccountMapper.updateAccount", act2);
    if (update1 == 1 && update2 == 1) {
        sqlSession.commit();
        sqlSession.close();
        return true;
    }
    sqlSession.rollback();
    sqlSession.close();
    return false;
}
```

```java
int update1 = sqlSession.update("AccountMapper.updateAccount", act1);
if (true) {
    throw new TransferException();
}
int update2 = sqlSession.update("AccountMapper.updateAccount", act2);
```

模拟一下异常，好像没什么问题，这样做也可以解决事务回滚，之后找机会补一下TreadLocal



## MyBatis三大对象生命周期

SqlSessionBuilder：执行结束即可被销毁，任务只是创建SqlSessionFactory，之后没有用处，这也是为什么不用单独的遍历存储SqlSessionBuilder

```java
sqlSessionFactory = new SqlSessionFactoryBuilder()
                    .build(Resources.getResourceAsStream("mybatis-config.xml"));
```



SqlSessionFactory：一旦创建就应该在应用运行期间一直存在，没有任何理由丢弃或新建另一个SqlSessionFactory实例对象，这也是为什么Util工具类封装的时候用静态代码块初始创建SqlSessionFactory对象

```java
static {
    try {
        sqlSessionFactory = new SqlSessionFactoryBuilder()
                .build(Resources.getResourceAsStream("mybatis-config.xml"));
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```

SqlSession：每个线程都应该有自己的SqlSession对象，SqlSession的实例不是线程安全的，因此不可以被共享，在finally语句块中关闭

明天进入javassist生产类的学习，它可以帮助我们生成Dao对象
