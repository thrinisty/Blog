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



## CRUD增删改查

在之前的时候，我们写死了插入数据和删除数据，而在实际使用的时候，我们需要的应该是代填项，等待我们插入对应数据，如JDBC中的?

```html
<mapper namespace="com.example.mapper.UserMapper">
    <insert id="insertValue">
        insert into t_car values
        (4, 301, "大众", 32.00, "2022-09-01", "燃油车");
    </insert>
</mapper>
```

再MyBatis中等效的写法应该是 #{} 代替JDBC中的 ? 

```html
<mapper namespace="com.example.mapper.UserMapper">
    <insert id="insertValue">
        insert into t_car values
        (null, #{}, #{}, #{}, #{}, #{});
    </insert>
</mapper>
```



### Insert

#### Map传入参数

另外的insert方法可以传入一个map对象，进行上述空格的赋值，在核心配置文件中的{}中填入key值

```html
<insert id="insertValue">
    insert into t_car values
    (null, #{k1}, #{k2}, #{k3}, #{k4}, #{k5});
</insert>
```

```java
@Test
public void testInsert() {
    SqlSession sqlSession = SqlSessionUtil.getSqlSession();
    Map<String, Object> map = new HashMap<String, Object>();
    map.put("k1", 401);
    map.put("k2", "比亚迪");
    map.put("k3", 42.00);
    map.put("k4", LocalDate.of(2010, 12, 10));
    map.put("k5", "电车");
    int i = sqlSession.insert("insertValue", map);
    sqlSession.commit();
    sqlSession.close();
}
```



#### Bean传入参数

我们构造一个JavaBean对象

```java
public class Car {
    private Integer id;
    private String car_num;
    private String brand;
    private double guide_price;
    private LocalDate produce_time;
    private String car_type;
    ...
}
```

在配置中的{}中传入Bean对象的属性名，其实是找getXxx方法获取参数

```html
<insert id="insertValue">
    insert into t_car values
    (null, #{car_num}, #{brand}, #{guide_price}, #{produce_time}, #{car_type});
</insert>
```

```java
@Test
public void testInsertBean() {
    SqlSession sqlSession = SqlSessionUtil.getSqlSession();
    Car car = new Car(null, "501", "奔驰", 21.00,
            LocalDate.of(2010, 12, 10), "新能源");
    int i = sqlSession.insert("insertValue", car);
    sqlSession.commit();
    sqlSession.close();
}
```



### Delete

核心配置，这里的参数只有一个，#{}可以随便写一个，例如id

```html
<delete id="deleteValueById">
    delete from t_car where id = #{id};
</delete>
```

传入id即可，这里会自动封装

```java
@Test
public void testDelete() {
    SqlSession sqlSession = SqlSessionUtil.getSqlSession();
    sqlSession.delete("deleteValueById", 1);
    sqlSession.commit();
    sqlSession.close();
}
```



### Update

和插入一致，传入car对象，通过用属性名，调用get方法获取参数

```html
<update id="updateValueById">
    update t_car set car_num = #{car_num}, brand = #{brand}, guide_price = #{guide_price}, produce_time = #{produce_time},car_type = #{car_type} where id = #{id};
</update>
```

```java
@Test
public void testUpdate() {
    SqlSession sqlSession = SqlSessionUtil.getSqlSession();
    Car car = new Car(3, "100", "奔驰_new", 21.00,
            LocalDate.of(2010, 12, 10), "旧能源");
    sqlSession.delete("updateValueById", car);
    sqlSession.commit();
    sqlSession.close();
}
```



### Select

#### 单个记录

在返回结果时需要用resultType指定映射的Bean对象全类名

```html
<select id="selectById" resultType="com.bean.Car">
    select * from t_car where id = #{id};
</select>
```

用一个Object对象返回结果对象

```java
@Test
public void testSelect() {
    SqlSession sqlSession = SqlSessionUtil.getSqlSession();
    Object object = sqlSession.selectOne("selectById", 3);
    System.out.println(object);
    sqlSession.commit();
    sqlSession.close();
}
```

```
Car{id=3, car_num='100', brand='奔驰_new', guide_price=21.0, produce_time=2010-12-10, car_type='旧能源'}
```

另外这里我用的Bean对象属性值，对应着数据表的字段（名字一致），如果名称不一致，我们需要使用AS通过别名映射Bean对象属性，才可以正确对Object赋值，否则为null



#### 所有记录

编写SQL语句

```html
<select id="selectAll" resultType="com.bean.Car">
    select * from t_car;
</select>
```

测试程序中通过selectList返回对象结果集合

```java
@Test
public void testSelectALL() {
    SqlSession sqlSession = SqlSessionUtil.getSqlSession();
    List<Object> list = sqlSession.selectList("selectAll");
    for (Object object : list) {
        System.out.println(object);
    }
    sqlSession.commit();
    sqlSession.close();
}
```



### namespace

我们在核心配置中引入两个Mapper配置，其中有id相同的两个SQL语句，我们要区别他们，在使用的时候通过namespace进行区分

```html
<mapper namespace="CarMapper">
```

```java
@Test
public void testSelectALL() {
    SqlSession sqlSession = SqlSessionUtil.getSqlSession();
    List<Object> list = sqlSession.selectList("CarMapper.selectAll");
    //全写，命名空间 + SQL ID，以区分不同命名空间下的相同id
    for (Object object : list) {
        System.out.println(object);
    }
    sqlSession.commit();
    sqlSession.close();
}
```



## MyBatis核心配置

### 多环境配置

在核心配置中，我们配置了两个环境，默认使用dev环境，还可以配置多个环境

```html
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
    <environment id="another">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/spring6?useSSL=false&amp;serverTimezone=UTC"/>
            <property name="username" value="root"/>
            <property name="password" value="654321"/>
        </dataSource>
    </environment>
</environments>
```

一个environment对应一个SqlSessionFactory对象

一个数据库对应一个SqlSessionFactory对象

在使用另一个环境的时候可以在build方法指定第二个参数，指定环境id

```java
sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"), "another");
```

这样得到的环境就是我们配置的名为another的环境，sql操作的数据库就是spring6数据库



### 数据源DataSource

通过数据源提供connection，也可以自己提供数据源，只要数据源实现数据源接口，提供connection对象

type指定的就是数据源对象，三选一：UNPOOLED、POOLED、JNDI

分别对应不使用连接池、使用连接池，使用外部第三方连接池

```html
<dataSource type="POOLED">
    <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;serverTimezone=UTC"/>
    <property name="username" value="root"/>
    <property name="password" value="654321"/>
</dataSource>
```

其中JNDI是一套规范，大部分的Web容器都实现了JNDI规范，是Java命名目录接口

使用不同的type其property一般不同，例如JNDI使用initial_context和data_source即可



### POOLED参数配置

使用连接池的时候可以使用很多的property参数，以达到更好的使用效率

之前在Druid连接池的配置文件中我们也使用到过

例如：

poolMaximumActiveConnections指定最多的使用连接对象的数量

poolTimeToWait指定每间隔多少秒打印日志，并且尝试获取下一个连接

poolMaximumCheckoutTime设置Connection的超时时间



在课程中CRUD结束后还有一个手写MyBatis的部分，时间紧张我就先跳过了，明天在Web项目中使用以下MyBatis，顺便复习Web的html，Servlet部分的知识
