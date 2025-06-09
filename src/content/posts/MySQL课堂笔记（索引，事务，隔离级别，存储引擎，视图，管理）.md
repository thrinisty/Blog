---
title: MySQL笔记
published: 2025-04-14
updated: 2025-04-14
description: '索引，事务，隔离级别，存储引擎，视图，管理'
image: ''
tags: [MySQL]
category: 'MySQL'
draft: false 
---

# MySQL

## 自增长

我们再插入记录的时候有的时候需要有的数据从1开始自动增长，我们这个时候就要运用到自增长

基本使用方式：在定义的时候设置auto_increment字段

```sql
字段名称 整形 primary key auto_increment
```

插入记录的时候填入null

```sql
CREATE TABLE dept ( 
`deptno` INT primary key auto_increment,
`dname` VARCHAR ( 30 ), 
`loc` VARCHAR ( 30 ) 
);
```

```sql
insert into dept values(null, 'ddd', 'beijing');
```

如果你需要自增需要从10开始，你也可以修改auto_increment设置

```sql
ALTER TABLE dept AUTO_INCREMENT = 10;
```

插入数据的时候子增值字段也可以不使用null，而插入一个具体的值

```sql
insert into dept values(50, 'ddd', 'beijing');
```

补充：设置自增步长

```sql
-- 1. 首先设置全局自增步长
SET @@auto_increment_increment = 10;

-- 2. 然后修改表的自增值
ALTER TABLE dept AUTO_INCREMENT = 1;
```



## 索引

索引可以在添加一定的代价后(占据更大大空间)，极大地增加查询（90%）的速度，而对于增删改（10%）会有一定的速率影响

### 基本使用方式

添加索引

```sql
CREATE [UNIQUE] INDEX 索引名称 on 表名(字段);
```

```sql
alter table 表名 add index 索引名称(字段);
```

删除索引

```sql
drop index 索引名称 on 表名;
```

删除主键索引

```sql
alter table dept drop primary key;
```



### 索引的类型

1.主键索引，主键自动的为主键添加索引（primary key）

2.唯一索引（unique），在创建表的时候，添加为唯一索引

3.普通索引（index）

4.全文索引（fulltext）在实际使用的时候一般不使用自带的全文索引往往切换为Solr或者ElasticSearch



显示来自于dept的索引

```sql
show index from dept;
```

```sql
show keys from dept;
```

```sql
show indexes from dept;
```



### 练习

为以下的表格添加主键索引

```sql
create table `order` (
`id` int,
`name` varchar(32),
`p_name` varchar(32),
`count` int
)
```

```sql
alter table `order` add unique index id_index (id);
```

```sql
create index id_index on `order`(id);
```



### 创建索引规则

1.较为频繁作为查询条件字段应该创建索引

```
例如：一个学校中的学生id
```

2.唯一性太差的字段不适合单独创建索引，即使查询频繁

```
例如：学生的性别
```

3.更新非常频繁的字段不适合创建索引

```
例如：学生账号的登录次数
```

4.不会出现在where子句中的字段不该创建索引

```
例如：学生信息表中的详细的家庭住址
```



## 事务

### 什么是事务？

事务用于保证数据的一致性，它由一组相关的dml语句组成，改组的dml语句要么全部执行，要么全部失败，例如在转账的时候就需要使用事务来处理，以保证数据的一致性



事务和锁

当执行事务操作的时候，mysql会在表上加锁，防止其他用户修改表的数据，这对于用户而言非常重要



### 引用示例

我们现在有一个运用场景，有如下的一张表格代表两个用户的银行账户信息

```sql
create table balance (
`id` int primary key,
`name` varchar(32),
`money` double
)

insert into balance values
(100,'tom',3000.00),
(200,'king',6000);
```



我们现在有如下需求：

将tom的100块钱转移到king的账户下

```sql
update balance set money = money-100 where id=100;
update balance set money = money+100 where id=200;
```

但是假如我们的数据库运行出现问题：第一条转账语句成功但是第二条失败，则tom转出的100块钱就凭空消失了，这个时候我们就需要运用到事务操作，将两条语句当为一个整体执行（要么全部执行成功，要么失败）



### 事务基本操作

#### start transaction

开始一个事务

#### savepoint

保存点名--设置保存点

#### rollback to

保存点名--回退事务

#### rollback

回退全部的事务

#### commit

提交事务，所有的操作生效，不可以回退



我们现在回到我们的实际案例上面

```sql
start transaction; --设置了事物的开始

savepoint a;  --保存点a
update balance set money = money-100 where id=100;--dml

savepoint b;  --保存点a
update balance set money = money+100 where id=200;--dml

ROLLBACK;  --回滚
commit;  --提交
select * from balance;
```

执行这样一段代码的话，数据不会被更改，因为进行了回滚，如果删除回滚在进行提交，执行成功，tom的100元给到了king



回退事务：当执行回退事物的时候，通过指定的保存点就可以回退到指定的点

提交事务：使用commit进行提交，执行了commit后就会确认事务的变化，结束事务，删除保存点，释放锁，数据生效。当使用commit语句结束事务以后，其他的会话可以查看到事物变化后的新数据



### 注意事项

1.如果不开始事务，默认情况下，dml操作是自动提交的，不可以进行回滚

2.如果开始一个事务，没有创建保存点，可以执行rollback操作，默认回滚到事务开始的状态

3.可以在事务还没有提交的时候，创建多个保存点

4.可以在事务还没有提交的时候，选择回退到哪个保存点

5.mysql事务机制需要innodb存储引擎，myisam不可以使用事务机制



### 事务的acid特性

（atomicity）原子性：原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生

（consistency）一致性：事务必须使数据库从一个一致性状态变换到另一个一致性状态

（isolation）隔离性：事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离

（durability）持久性：持久性是指一个事务一旦被提交，他对于数据库中的数据的改变就是永久的，接下来即使数据库发生故障可而不应该对其有任何的影响



## 隔离级别

脏读：当一个事务读取另一个事务尚未提交的修改时，产生脏读

不可重复读：同一查询在同一事物中多次进行，由于其他提交事务所做的修改或者删除，每次返回结果不同的结果集，产生不可重复读

幻读：同一查询在同一事物中多次进行，由于其他提交事务的插入操作，每次返回不同的结果集，产生了幻读



### 事务隔离表格

事务隔离级别：MySQL隔离级别定义了事物与事务之间的隔离程度

| 隔离级别 | 丢失修改 | 脏读     | 不可重复读 | 幻读     |
| :------: | -------- | -------- | ---------- | -------- |
| 读未提交 | 不会发生 | 可能发生 | 可能发生   | 可能发生 |
| 读已提交 | 不会发生 | 不会发生 | 可能发生   | 可能发生 |
| 可重复读 | 不会发生 | 不会发生 | 不会发生   | 可能发生 |
| 可串行化 | 不会发生 | 不会发生 | 不会发生   | 不会发生 |



### Mysql中解决幻读

Mysql中的默认隔离级别是可重复读，但是在使用innodb引擎的时候不会发生幻读，原因如下

InnoDB 存储引擎通过多版本并发控制(MVCC, Multi-Version Concurrency Control)机制，在"可重复读"(Repeatable Read)隔离级别下有效解决了幻读问题。

```sql
普通SELECT操作(快照读)
通过ReadView判断哪些版本对当前事务可见
总是读取事务开始时的一致性快照
其他事务的插入操作不会影响当前事务的查询结果

当前读操作
对于SELECT ... FOR UPDATE、UPDATE、DELETE等操作：
InnoDB会加Next-Key Lock(记录锁+间隙锁)
防止其他事务在查询范围内插入新记录
从而彻底解决幻读问题
```

```sql
START TRANSACTION;
SELECT * FROM t WHERE id > 1; -- 看到3,5
-- 此时事务B插入id=4并提交
SELECT * FROM t WHERE id > 1; -- 仍然只看到3,5(快照读)
SELECT * FROM t WHERE id > 1 FOR UPDATE; -- 会看到3,4,5(当前读)
COMMIT;
```

在可重复读隔离级别下，普通SELECT不会看到事务B插入的id=4，而加锁的SELECT会看到并锁定这些记录。



### 读已提交和可重复读

| 特性               | 读已提交(Read Committed) | 可重复读(Repeatable Read)    |
| :----------------- | :----------------------- | :--------------------------- |
| **数据可见性规则** | 总是看到最新已提交的数据 | **看到事务开始时的数据快照** |
| **锁机制**         | 语句级锁(执行完立即释放) | 事务级锁(保持到事务结束)     |
| **不可重复读**     | 可能发生                 | 不会发生                     |
| **性能**           | 较高                     | 中等                         |



### 可重复读和可串行化

| 特性             | 可重复读(Repeatable Read) | 可串行化(Serializable)    |
| :--------------- | :------------------------ | :------------------------ |
| **并发控制原理** | MVCC(多版本并发控制)为主  | 严格的二阶段封锁协议      |
| **读取行为**     | 快照读(非锁定读)          | 所有读自动转为锁定读(S锁) |
| **写入行为**     | 需要时加X锁               | 总是加X锁                 |
| **锁范围**       | 仅锁定实际访问的行和间隙  | 更大范围的锁(更保守)      |
| **性能**         | 高并发，读不阻塞写        | 低并发，读写相互阻塞      |



### 常用代码

#### 查看隔离级别

可以通过以下代码查看数据库设置的隔离级别

MySQL 5.7及之前版本

```sql
select @@tx_isolation;
```

MySQL 8.0及之后版本

查看系统当前隔离级别

```sql
SELECT @@transaction_isolation;
```

```sql
+-------------------------+
| @@transaction_isolation |
+-------------------------+
| REPEATABLE-READ         |
+-------------------------+
```

可见Mysql的默认隔离级别为可重复读



#### 设置隔离级别

设置当前会话隔离级别

```sql
SET SESSION TRANSACTION ISOLATION LEVEL 隔离级别;
```

设置系统当前隔离级别

```sql
SET global TRANSACTION ISOLATION LEVEL 隔离级别;
```

##### 读未提交

```sql
SET SESSION TRANSACTION ISOLATION LEVEL read uncommitted;
```

##### 读已提交

```sql
SET SESSION TRANSACTION ISOLATION LEVEL read committed;
```

##### 可重复读

```sql
SET SESSION TRANSACTION ISOLATION LEVEL repeatable read;
```

##### 可串行化

```sql
SET SESSION TRANSACTION ISOLATION LEVEL serializable;
```



## 存储引擎

1.MySQL的表类型由存储引擎决定，主要包括MYISAM，InnoDB，Memory

2.MySQL数据表只要支持六种类型：CSV，Memory，ARCHIVE，MRG_MYISAM，MYISAM，InnoDB

3.这两种存储又分为两种：一类是事务安全型，例如InnoDB，其余为第二类，称为非事务安全型



在MySQL中可以使用如下的指令查看存储引擎

```sql
SHOW ENGINES;
```

### 引擎比较

接下来来比较InnoDB、Myisam、Memory、Archive之间的优缺点：

| 特性     | InnoDB     | MyISAM   | Memory       | Archive      |
| :------- | :--------- | :------- | :----------- | :----------- |
| 事务支持 | 支持       | 不支持   | 不支持       | 不支持       |
| 锁定级别 | 行级锁     | 表锁     | 表锁         | 行锁(仅插入) |
| 外键支持 | 支持       | 不支持   | 不支持       | 不支持       |
| 存储位置 | 磁盘       | 磁盘     | 内存         | 磁盘         |
| 压缩能力 | 中等       | 低       | 无           | 极高         |
| 崩溃恢复 | 优秀       | 差       | 无(数据丢失) | 中等         |
| 全文索引 | 5.6+支持   | 支持     | 不支持       | 不支持       |
| 适用场景 | 高并发OLTP | 读密集型 | 临时/缓存    | 归档存储     |



### 小结

1.MyISAM不支持事务、也不支持外键、但是访问速度极快、对于事物完整性没有要求

2.InnoDB存储引擎提供了具有提交、回滚和崩溃恢复能力的事务安全、但是相比于MyISAM存储引擎，InnoDB写的处理效率差一些并且会占用更多的从磁盘空间以保留数据和索引

3.Memory存储引擎使用存在内存中的内容来创建表，每个Memory表只实际对应一个磁盘文件，该类型的表访问速度极快，因为数据是存放在内存中的，并且默认使用了Hash索引，但是MySQL服务关闭，表中的数据就会丢失，表的结构还在



### 使用案例

修改存储引擎

```sql
alter talble 表名 engine = 存储引擎;
```

```sql
alter table balance engine = innodb;
```

创建一个MyISAM存储引擎的表，无法使用事务

```sql
create table balance (
`id` int primary key,
`name` varchar(32),
`money` double
) ENGINE MYISAM;
```

创建一个Memory存储引擎的表，支持Hash索引，执行速度很快

```sql
create table balance (
`id` int primary key,
`name` varchar(32),
`money` double
) ENGINE Memory;
```



### 总结

1.如果应用不需要事务，处理的只是基本的增删改查，那么MyISAM是绝佳的选择，速度快

2.需要事务支持，就选择InnoDB

3.Memory存储引擎将数据存在内存中，由于没有磁盘的IO等待，速度非常快，但数据不持久



## 视图

我们有的时候需要将表中的某几个字段显示使用（有权限需要，不能够每个人都使用select查询整一张表格），我们可以将这几个字段拿出来做成一个视图，还可以将视图的查看权限赋予某个用户

### 基本概念

视图是一个虚拟表，其内容由查询定义，同真实的表一样，视图包含列，数据来自于对应的真实表（基表）

1.视图是根据基表（可以多个基表）来创建的，视图是虚拟的表

2.视图也有列，数据来自于基表

3.通过修改视图可以修改基表的数据

4.基表的改变也会影响到视图的数据



### 基本使用

创建视图

```sql
create view 视图名称 as select语句;
```

更新视图

```sql
alter view 视图名称 as select语句;
```

显示视图是如何创建的（select语句）

```sql
show create view 视图名;
```

删除视图

```sql
drop view 视图1,视图2...;
```

我们要对于员工表创建一个名为emp_view的视图（数据包含empno,ename,job,deptno）

```sql
create view emp_view as 
select empno,ename,job,deptno from emp;
```

我们现在来查看一下这个视图

```sql
select * from emp_view;
```

```sql
+-------+--------+----------+--------+
| empno | ename  | job      | deptno |
+-------+--------+----------+--------+
|  7396 | SMITH  | CLERK    |     20 |
|  7499 | ALLEN  | SALESMAN |     30 |
|  7596 | WARD   | SALESMAN |     30 |
|  7696 | JOOES  | MANAGER  |     20 |
|  7796 | MARTIN | SALESMAN |     30 |
|  7896 | BLAKE  | MANAGER  |     30 |
+-------+--------+----------+--------+
```

删除视图

```sql
drop view emp_view;
```



例题：针对emp，dept，salgrade 三张表格创建一个emp_view视图，可以显示雇员编号，雇员名称，雇员部门名称，薪水级别

相关的select语句

```sql
select empno,ename,dname,grade from emp, dept, salgrade 
where emp.deptno = dept.deptno
and sal between losal and hisal;
```

制作视图

```sql
create view emp_view as
select empno,ename,dname,grade from emp, dept, salgrade 
where emp.deptno = dept.deptno
and sal between losal and hisal;
```



### 视图细节

1.创建视图后，到数据库看，对应的视图只有一个视图结构文件(视图名.frm)

2.视图和基表的修改相互影响

```sql
update emp_view set deptno = 40 where ename='SMITH';
```

```sql
update emp set deptno = 20 where ename='SMITH';
```

3.视图中可以再次使用视图



### 使用场景

安全：一些数据表有着重要的信息，某些字段需要保密，不可以让用户直接看到，这个时候我们就可以创建一个视图，在这个视图中保留安全的字段，而不包含保密的字段

性能：关系数据库常常会分表存储，使用外键建立这一些表的关系，这个时候，数据库查询通常会用到join连接，这么做不但麻烦，效率也比较低，如果建立一个视图，将相关表的字段组合在一起，就可以避免使用join查询数据

灵活：如果系统中有一张旧的表，这张表由于设计的问题，即将被废弃，然而，很多表都是基于这一张表，不方便修改，这个时候就可以建立一张视图，视图中的数据直接映射到新建的表，这样就可以做很少的改动，也达到了升级数据表的目的



## Mysql管理

### user表

mysql中的用户，都存储在系统数据库mysql中的user表

其中有一些重要的字段

host：允许登录的位置，localhost表示该用户只允许本机登录，也可以指定ip

user：用户名称

authentication_string：密码，通过mysql的password()函数加密后的密码

```sql
select host,user,authentication_string from user;
```

```sql
+-----------+------------------+------------------------------------------------------------------------+
| host      | user             | authentication_string                                                  |
+-----------+------------------+------------------------------------------------------------------------+
| localhost | mysql.infoschema | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.session    | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.sys        | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | root             | $A$005$ xgY]U4ieS
8Q+#hB9QVmRfma9kzzSPEIJ1FgvgP7VMuobh4V.rZD0mc7FD  |
+-----------+------------------+------------------------------------------------------------------------+
```



### 用户管理

当做项目开发的时候，Mysql数据库管理人员可以根据不同的开发人员，赋给相应的Mysql操作权限，供人员使用

#### 创建用户

```sql
create user 'thrinisty' @'localhost' identified by '123456';
```

```sql
select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
| localhost | root             |
| localhost | thrinisty        |
+-----------+------------------+
```

我们在Navicat中可以用新的用户连接数据库

![111](../images/111.png)

可以看到我们新用户表和root用户的表不太一样

![112](../images/112.png)

这是因为不同的数据库用户，登录到DBMS，根据相应的权限，可以操作的数据库，数据对象（表、视图、触发器）都不一样



#### 删除用户

```sql
drop user '用户名' @'允许登录的位置'
```



#### 修改密码

注意以下修改方式是5.7.5版本以下的

```sql
set password = password('密码');--修改自身密码
```

```sql
set password for '用户名' @'登录位置' = password('密码');
--修改他人的密码，需要有用户密码的权限
```

我用的8.0版本需要使用

```sql
ALTER USER USER() IDENTIFIED BY '654321';--修改自身密码
```

```sql
ALTER USER 'username'@'hostname' IDENTIFIED BY '654321';
--修改他人的密码，需要有用户密码的权限
```



### 权限管理

MySQL中有非常多的权限可以赋予用户，我们来使用以下常见的权限

#### 常见的权限

| 全选           | 说明                              |
| -------------- | --------------------------------- |
| GRANT OPTION   | 允许授予权限                      |
| ALL PRIVIEGES  | 设置除GRANT OPTTION之外的简单权限 |
| ALTER          | 允许使用 ALTER TABLE              |
| CREATE         | 允许使用 CREATE TABLE             |
| CREATE USER    | 允许使用增删改用户、移除所有权限  |
| CREATE VIEW    | 允许使用CREATE VIEW               |
| DELETE         | 允许使用DELETE                    |
| DROP           | 允许使用DROP TABLE                |
| INDEX          | 允许创建删除索引                  |
| INSERT         | 允许使用INSERT插入                |
| SELECT         | 允许使用查询                      |
| SHOW DATABASES | 允许显示所有数据库                |
| USAGE          | 无权限                            |



#### 基本语法

赋权指令

```sql
grant 权限列表 on 库.对象名 to 
'用户名'@'登录位置' [identified by '密码'];
```

回收权限

```sql
revoke 权限列表 on 库.对象名 from 
'用户名'@'登录位置'
```

刷新权限

```sql
FLUSH PRIVILEGES;
```

赋给thrinisty全部表全部对象的all权限

```sql
grant all on *.* to 'thrinisty'@'localhost';
```



### 管理细节

1.在创建用户的时候，如果不指定host，则为%，表示所有ip都有链接的权限

```sql
create user jack;
```

2.在删除用户的时候，如果host不是%，需要明确指定‘用户’@‘host值’
