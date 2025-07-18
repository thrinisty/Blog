---
title: 用户中心项目笔记（MyBatisPlus，注册，登录，记录状态）
published: 2025-07-03
updated: 2025-07-03
description: '用户中心项目后端开发，SpringBoot整合'
image: ''
tags: [Project]
category: 'Project'
draft: false 
---

# 用户中心开发笔记

## MyBatisPlus

引入相关依赖

mybatisPlus依赖

```html
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.12</version>
</dependency>
```

lombok依赖，用于自动生成get set toString方法

```html
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
</dependency>
```

配置文件

```yaml
spring:
  application:
    name: user-center
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/usermanager
    username: root
    password: 654321
```



Bean对象的设置

```java
@Data
@TableName("`user`")
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```



创建UserMapper的接口，继承于BaseMapper<User>，这个父类接口中存在很多现成的SQL操作方法供以使用，使得我们不需要手动写SQL语句

```java
public interface UserMapper extends BaseMapper<User> {

}
```



在引导类中使用MapperScan扫描UserMapper

```java
@SpringBootApplication
@MapperScan("com.nwpu.backend.mapper")
public class BackendApplication {
    public static void main(String[] args) {
       SpringApplication.run(BackendApplication.class, args);
    }
}
```



测试程序

```sql
DROP TABLE IF EXISTS `user`;

CREATE TABLE `user`
(
    id BIGINT NOT NULL COMMENT '主键ID',
    name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
    age INT NULL DEFAULT NULL COMMENT '年龄',
    email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
    PRIMARY KEY (id)
);

DELETE FROM `user`;

INSERT INTO `user` (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```



```java
@Test
void testUserMapper() {
    List<User> users = userMapper.selectList(null);
    for (User user : users) {
       System.out.println(user);
    }
}
```

```
User(id=1, name=Jone, age=18, email=test1@baomidou.com)
User(id=1, name=Jone, age=18, email=test1@baomidou.com)
User(id=2, name=Jack, age=20, email=test2@baomidou.com)
User(id=3, name=Tom, age=28, email=test3@baomidou.com)
User(id=4, name=Sandy, age=21, email=test4@baomidou.com)
User(id=5, name=Billie, age=24, email=test5@baomidou.com)
```



## 数据库表格设计

```sql
CREATE TABLE `usermanager`.`user`  (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `username` varchar(100) NULL COMMENT '用户名称',
  `user_account` varchar(255) NULL COMMENT '用户账号',
  `avatar_url` varchar(255) NULL COMMENT '用户头像地址',
  `gender` tinyint NULL COMMENT '用户性别',
  `user_password` varchar(255) NOT NULL COMMENT '用户密码',
  `phone` varchar(255) NULL COMMENT '用户电话',
  `email` varchar(255) NULL COMMENT '用户邮箱',
  `user_status` tinyint ZEROFILL NULL COMMENT '用户状态',
  `createtime` datetime DEFAULT CURRENT_TIMESTAMP	NULL COMMENT '创建时间',
  `updatetime` datetime DEFAULT CURRENT_TIMESTAMP NULL on update CURRENT_TIMESTAMP COMMENT '更新时间',
  `is_delete` tinyint ZEROFILL NULL COMMENT '是否删除',
  PRIMARY KEY (`id`)
);
```



User对象创建

```java
@Data
public class User implements Serializable {
    private Long id;
    private String username;
    private String userAccount;
    private String avatarUrl;
    private Integer gender;
    private String userPassword;
    private String phone;
    private String email;
    private Integer userStatus;
    private Date createtime;
    private Date updatetime;
    private Integer isDelete;
}
```



## 注册逻辑实现

1.用户在前端输入账户密码，以及校验码

2.校验账户密码：

​	账户不少于4位

​	密码不小于8位

​	账户不能重复

3.对密码进行加密（密码不能直接存入到数据库）

4.向用户数据库插入数据



设置User业务层的接口，extends IService<User>是预设了一些常用的方法

```java
public interface UserService extends IService<User> {

    /**
     * 用户注册
     * @param userAccount 用户账号
     * @param userPassword 用户密码
     * @param checkPassword 确认密码
     * @return 新用户ID
     */
    long userRegister(String userAccount, String userPassword, String checkPassword);
}
```

实现上，继承于ServiceImpl<UserMapper, User> 可以默认实现预设的逻辑

- **`M`（Mapper 类型）**：指定操作数据库的 Mapper 接口（这里是 `UserMapper`）。
- **`T`（实体类型）**：指定数据库对应的实体类（这里是 `User`）。

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    @Override
    public long userRegister(String userAccount, String userPassword, String checkPassword) {
        return 0;
    }
}
```

这里我们为了方便引入一个commons-lang3依赖

```html
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.17.0</version>
</dependency>
```

密码加密，这里引入一个加密相关的依赖

```html
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-crypto</artifactId>
</dependency>
```

工具类的封装

```java
public class PasswordUtils {
    private static final BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
    // 加密密码
    public static String encrypt(String rawPassword) {
        return encoder.encode(rawPassword);
    }
    // 校验密码
    public static boolean matches(String rawPassword, String encodedPassword) {
        return encoder.matches(rawPassword, encodedPassword);
    }
}
```

测试一下

```java
@Test
public void testEncode() {
    String password = "feierling";
    String encodePwd = PasswordUtils.encrypt(password);
    System.out.println(encodePwd);
    System.out.println(PasswordUtils.matches("feierling", encodePwd));
}
```

```
$2a$10$c31FHZvkQNmUb5K3qByys.aw8igBVHC7nRkg1Rb2dmK9//PIa9QIi
true
```

:::warning

在 **BCrypt** 加密算法中，**即使相同的密码，每次加密后的哈希值也会不同**

:::

注册代码实现

```java
@Override
public long userRegister(String userAccount, String userPassword, String checkPassword) {
    //使用工具类传入参数列表，都不为空
    if (StringUtils.isAnyBlank(userAccount, userPassword, checkPassword)) {
        return -1;//不能为空
    }
    if (userAccount.length() < 4) {
        return -2;//账户不能小于4
    }
    if (userPassword.length() < 8 || checkPassword.length() < 8) {
        return -3;//密码不能小于8
    }
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.eq("user_account", userAccount);
    long count = this.count(queryWrapper);
    if (count > 0) {
        return -4;//不能用相同的账户
    }
    Pattern pattern = Pattern.compile("[^a-zA-Z0-9\\u4e00-\\u9fa5]");
    if (pattern.matcher(userAccount).find()) {
        return -5;//账户名不能为特殊字符
    }
    if (!userPassword.equals(checkPassword)){
        return -6;//密码不相同
    }

    //加密
    String encodePwd = PasswordUtils.encrypt(userPassword);
    User user = new User();
    user.setUserAccount(userAccount);
    user.setUserPassword(encodePwd);

    //向数据库中插入数据
    boolean saveResult = this.save(user);
    if (!saveResult) {
        return -7;//插入失败
    }
    return user.getId();
}
```



## 登录逻辑实现

接收参数：用户账户名称，密码

请求类型：POST

请求体：JSON格式

返回值：用户信息（脱敏）



### 逻辑

校验账户密码是否合法

账户是否在数据库中存在

将用户态记录，将其存到服务器上（Session、Cookie）

返回用户信息



### 代码实现

```java
@Override
public User userLogin(String userAccount, String userPassword) {
    //使用工具类传入参数列表，都不为空
    if (StringUtils.isAnyBlank(userAccount, userPassword)) {
        return null;//账户密码不能为空
    }
    if (userAccount.length() < 4) {
        return null;//账户不能小于4
    }
    if (userPassword.length() < 8) {
        return null;//密码不能小于8
    }

    Pattern pattern = Pattern.compile("[^a-zA-Z0-9\\u4e00-\\u9fa5]");
    if (pattern.matcher(userAccount).find()) {
        return null;//账户名不能为特殊字符
    }

    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.eq("user_account", userAccount);
    User user = this.getOne(queryWrapper);
    if (user == null) {
        return null;//用户不存在
    }
    Boolean isPasswordValid = PasswordUtils.matches(userPassword, user.getUserPassword());
    if (!isPasswordValid) {
        return null;//密码不正确
    }

    user.setUserPassword(null);//信息脱敏
    return user;
}
```



## 记录用户登录状态

连接服务器后，得到一个session状态，返回给前端

登陆成功后，得到了登陆成功的session，并且给该session设置一些值，返回给前端一个设置cookie的“命令”

前端接收到后端的命令后，设置cookie，保存到浏览器内

前端再次请求后端的时候，在请求头中带上cookie去请求

后端拿到前端传过来的cookie，找到对应的session

后端从session中可以取出基于该session存储的变量（用户的登录信息，登录名称）



## Controller

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping("/register")
    public Long userRegister(@RequestBody UserRegisterRequest userRegisterRequest) {
        if(userRegisterRequest == null) {
            return null;
        }
        String userAccount = userRegisterRequest.getUserAccount();
        String userPassword = userRegisterRequest.getUserPassword();
        String checkPassword = userRegisterRequest.getCheckPassword();
        return userService.userRegister(userAccount, userPassword, checkPassword);
    }

    @PostMapping("/login")
    public User userLogin(@RequestBody UserLoginRequest userLoginRequest, HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) {
        if(userLoginRequest == null) {
            return null;
        }
        String userAccount = userLoginRequest.getUserAccount();
        String userPassword = userLoginRequest.getUserPassword();
        return userService.userLogin(userAccount, userPassword, httpServletRequest);
    }
}
```
