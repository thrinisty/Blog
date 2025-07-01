---
title: SpringBoot笔记（原理分析，监控，项目部署）
published: 2025-06-26
updated: 2025-06-26
description: ''
image: ''
tags: [SpringBoot]
category: 'Frame'
draft: false 
---

# SpringBoot笔记

Spring高级部分包括以下三个部分

SpringBoot原理分析  SpringBoot监控  SpringBoot项目部署



## 对象创建原理

导入pom创建IoC对象

```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

在SpringBoot的引导类中，调用的run方法其实会返回一个context的实例对象，通过这个实例对象，我们可以用key从IoC容器中取出对应的Bean对象，我们导入了redis的pom坐标，context中就会存在这样的一个redisTemplate Bean对象

```java
@SpringBootApplication
public class Springboot008Application {
    public static void main(String[] args) {
       ConfigurableApplicationContext context = SpringApplication.run(Springboot008Application.class, args);
       Object redisTemplate = context.getBean("redisTemplate");
       System.out.println(redisTemplate);
    }
}
```

```
org.springframework.data.redis.core.RedisTemplate@27ffd9f8
```



### Condition

Condition是在Spring4.0增加条件判断功能，通过这个Condition功能可以实现选择性的创建Bean的操作

使用步骤

1.自定义类实现Condition接口

2.重写matches方法，在其中进行逻辑判断

3.在@Bean注解纳入管理的类上添加 @Conditional(自定义实现类.class)



在config软件包下的UserConfig类中用@Bean将User纳入Bean容器的管理

```java
@Configuration
public class UserConfig {
    @Bean
    public User user() {
        return new User();
    }
}
```

通过context对象getBean一样可以获取到user对象



接下来我们通过@Conditional(ClassCondition.class)指定一个实现Condition的ClassCondition类

```java
@Configuration
public class UserConfig {
    @Bean(name = "user")
    @Conditional(ClassCondition.class)
    public User user() {
        return new User();
    }
}
```

在这个类中我们在matches方法返回false，这代表不进行user对象加入IoC容器进行管理

```java
public class ClassCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return false;
    }
}
```

我们再次尝试获取user对象的时候就会发生报错，没有获取到user的bean对象，我们可以通过这个方法实现是否创建对应bean对象的逻辑

例如：

我们现在有一个要求，导入Jedis坐标后创建User的Bean对象，否则不纳入IoC管理，在没有获取到Jedis类的时候返回false，从而实现对应Bean的创建与否

```java
public class ClassCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        boolean flag = true;
        try {
            Class.forName("redis.clients.jedis.Jedis");
        } catch (ClassNotFoundException e) {
            flag = false;
        }
        return flag;
    }
}
```

将类对象用@Conditional注解配置到Bean的创建上

```java
@Configuration
public class UserConfig {
    @Bean(name = "user")
    @Conditional(ClassCondition.class)
    public User user() {
        return new User();
    }
}
```

这个时候只有导入了jedis坐标的时候才会创建user的Bean对象放入到IoC容器中

```html
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

matches(ConditionContext context, AnnotatedTypeMetadata metadata)方法

通过context.getEnviroment可以获取environment对象从而获取配置值

metadata是注解元对象，可以获取注解定义的属性值

SpringBoot是通过类似这样的方式根据有无对应字节码，选择性创建Bean对象



而SpringBoot提供的常用条件注解如下：

ConditionalOnProperty：判断配置文件中有对应属性和值

ConditionalOnClass：判断环境中有对应字节码文件

ConditionalOnMissingBean：判断环境中没有对应Bean



## 切换内置Web服务器

SpringBoot的Web环境中默认使用Tomcat作为内置服务器，总共SpringBoot有4种内置服务器供选择，可以方便的进行环境下切换

通过两步完成

1.排除Tomcat依赖

2.引入jetty依赖



## Enable注解

Spring不可以直接获取其他工程中的Bean对象

@ComponentScan的扫描范围是当前引导类所在的包以及其子包，所以无法加载到其他工程下的Bean对象，我们对于这种情况有两种解决方法

```java
package com.learn.springboot008.config;

@Configuration
public class UserConfig {
    @Bean(name = "user")
    @Conditional(ClassCondition.class)
    public User user() {
        return new User();
    }
}
```



第一种：用@ComponentScan注解扫描其他工程的对应配置软件包

```java
package com.learn.springboot009;
@SpringBootApplication
@ComponentScan("com.learn.springboot008.config")//扫描其他包
public class Springboot009Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Springboot009Application.class, args);
        User user = context.getBean("user", User.class);
        System.out.println(user);
    }
}
```

将其的Bean加入到IoC容器中即可，但是这种方案需要记住第三方包的软件包路径，不是非常方便



第二种：使用@Import注解，被Import注解导入的类，都会被IoC容器创建并纳入管理

```java
@SpringBootApplication
//@ComponentScan("com.learn.springboot008.config")
@Import(UserConfig.class)
public class Springboot009Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Springboot009Application.class, args);
        User user = context.getBean("user", User.class);
        System.out.println(user);
    }
}
```



第三种：在第三方对Import注解进行封装为EnableUser，在开发工程直接使用EnableUser注解

```java
package com.learn.springboot008.config;

import org.springframework.context.annotation.Import;

import java.lang.annotation.*;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(UserConfig.class)
public @interface EnableUser {
}
```

```java
@SpringBootApplication
//@ComponentScan("com.learn.springboot008.config")
//@Import(UserConfig.class)
@EnableUser
public class Springboot009Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Springboot009Application.class, args);
        User user = context.getBean("user", User.class);
        System.out.println(user);
    }
}
```



## Import注解

@Enable底层依赖于@Import注解导入一些类，使用@Import导入的类会被Spring加载到IoC容器中，而@Import提供四种用法

1.导入Bean

:::warning

Spring 默认会使用类的简单名称（首字母小写）作为 Bean 的名称，但在直接 `@Import` 类的情况下，Bean 的名称实际上是全限定类名（fully qualified class name），而不是简单的 "user"

:::

或者也可以直接通过类获取User user = context.getBean(User.class);

```java
@SpringBootApplication
//@ComponentScan("com.learn.springboot008.config")
//@Import(UserConfig.class)
//@EnableUser
@Import(User.class)
public class Springboot009Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Springboot009Application.class, args);
        User user = context.getBean("com.learn.springboot008.bean.User" ,User.class);
        //User user = context.getBean(User.class);
        System.out.println(user);
    }
}
```



2.导入配置类

UserConfig

```java
@Configuration//这个注解也可以不加
public class UserConfig {
    @Bean(name = "user")
    public User user() {
        return new User();
    }

    @Bean(name = "student")
    public Student student() {
        return new Student();
    }
}
```

在主工程种获取对象

```java
@SpringBootApplication
@Import(UserConfig.class)
public class Springboot009Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Springboot009Application.class, args);
        User user = context.getBean(User.class);
        System.out.println(user);
        Student student = context.getBean("student", Student.class);
        System.out.println(student);
    }
}
```



3.导入ImportSelector实现类，一般用一加载配置文件中的类

MyImportSelector，实现一个继承于ImportSelector的类，在selectImports方法中返回类的全限定类名的字符串

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"com.learn.springboot008.bean.User", "com.learn.springboot008.bean.Student"};
    }
}
```

在主工程中的引导类用如下注解导入

```
@Import(MyImportSelector.class)
```

 

4.导入ImportBeanDefinitionRegistrar实现类

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        BeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(User.class).getBeanDefinition();
        registry.registerBeanDefinition("user", beanDefinition);
    }
}
```

```java
@Import(MyImportSelector.class)
public class Springboot009Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Springboot009Application.class, args);
        User user = context.getBean(User.class);
        System.out.println(user);
    }
}
```



## SpringBoot监听

SpringBoot的监听机制，其实就是对于java提供的监听机制的封装

Java中的事件监听机制定义了以下几个角色：

Event事件：继承于EvenObject类的对象

Source事件源：任意对象Object

Listener监听器：实现EventListener接口的对象



SpringBoot在项目启动的时候，会对几个监听器进行回调，我们可以实现这些监听器接口，在项目启动时完成一些操作

ApplicationContextInitializer、SpringApplicationRunListener、CommandLineRunner、ApplicationRunner



后两种只需要实现接口注册到Spring容器中即可被使用

CommandLineRunner、ApplicationRunner当项目启动后被执行，例如使用Redis的时候，将缓存进行预热

其中的args是我们传递的参数，两个用一个即可

```java
@Component
public class MyCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("MyCommandLineRunner");
    }
}
```

```java
@Component
public class MyApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("MyApplicationRunner");
    }
}
```

```
MyCommandLineRunner
MyApplicationRunner
```



而前两种需要进行配置

在META-INF/spring.factories配置

ApplicationContextInitializer设置键为全限定类名，值为我们的实现类名

```properties
org.springframework.context.ApplicationContextInitializer=com.learn.springboot010.listener.MyApplicationContextInitializer
```

```
MyApplicationContextInitializer
2025-06-26T11:38:42.231+08:00  INFO 24284 --- [springboot-010] [           main] c.l.s.Springboot010Application           : Starting Springboot010Application using Java 17.0.15 with PID 24284 ...
MyApplicationRunner
MyCommandLineRunner
```

用于没有准备IoC容器的时候进行一些资源的检查

```properties
org.springframework.context.ApplicationContextInitializer=com.learn.springboot010.listener.MyApplicationContextInitializer
org.springframework.boot.SpringApplicationRunListener=com.learn.springboot010.listener.MySpringApplicationRunListener
```

```
starting 项目启动中...
environmentPrepared 环境对象开始准备...

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.5.3)

MyApplicationContextInitializer
contextPrepared 上下文对象开始准备...
2025-06-26T11:42:28.123+08:00  INFO 21740 --- [springboot-010] [           main] c.l.s.Springboot010Application           : Starting Springboot010Application using Java 17.0.15 with PID 21740 (C:\Users\71460\Desktop\Java\Spring\SpringBootLearn\springboot-010\target\classes started by 71460 in C:\Users\71460\Desktop\Java\Spring\SpringBootLearn)
2025-06-26T11:42:28.127+08:00  INFO 21740 --- [springboot-010] [           main] c.l.s.Springboot010Application           : No active profile set, falling back to 1 default profile: "default"
contextLoaded 上下文对象准备完毕
2025-06-26T11:42:28.877+08:00  INFO 21740 --- [springboot-010] [           main] c.l.s.Springboot010Application           : Started Springboot010Application in 1.434 seconds (process running for 2.271)
started 项目启动完毕
MyApplicationRunner
MyCommandLineRunner
ready 准备完成

```



## SpringBoot监控

SpringBoot自带监控功能Actuator，导入pom坐标即可使用

可以检查SpringBoot运行状态，或者一些第三方的数据库的运行状态

一般有info 和 health属性，还可以将监控的所有endpoint暴露出来

```
management.endpoint.health.show-details=always
management.endpoints.web.exposure.include=*
```

有Spring Boot Admin开源项目用于管理监控SpringBoot应用程序

其中涉及到了admin-server和admin-client



admin-server

1.创建admin-server模块

2.导入admin-starter-server

3在引导类上启动监控共功能@EnableAdminServer

```java
@EnableAdminServer
@SpringBootApplication
public class AdminServerApplication {

    public static void main(String[] args) {
       SpringApplication.run(AdminServerApplication.class, args);
    }

}
```

在配置中配置下端口9000



admin-client

1.创建admin-client模块

2.导入admin-starter-client

3.配置相关信息：server地址等

4.启动server和client服务，访问server

```
spring.application.name=admin-client

spring.boot.admin.client.url=http://localhost:9000
management.endpoints.web.exposure.include=*
```

之后通过http://localhost:9000/ 访问server即可在网页中显示可视化信息



## SpringBoot部署

SpringBoot项目开发完毕后支持两种方式部署到服务器中，jar包，或者war包

之前我们尝试过打包为jar包，通过java指令启动，我们现在尝试打成war包



在pom文件中引入war包的依赖

在引导类中继承SpringBootServletInitializer，重写方法将引导类的类放入builder.sources()返回

```java
@SpringBootApplication
public class Springboot004Application extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(Springboot004Application.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Springboot004Application.class);
    }
}
```

将war包放入web-app启动Tomcat即可访问
