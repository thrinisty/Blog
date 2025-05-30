---
title: Spring笔记（手写Spring）
published: 2025-05-30
updated: 2025-05-30
description: '简单地手写一个Spring框架'
image: ''
tags: [Spring]
category: 'Frame'
draft: false 
---

# Spring笔记

## 回顾反射

写一个类，通过反射执行方法

```java
package com.thrinisty.bean;

public class User {
    private String name;
    private int age;

    public User() {}

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

DI核心核心代码实现：通过class类名和filed属性名称获取对象以及字段赋值

```java
public class Main {
    public static void main(String[] args) throws Exception{
        String className = "com.thrinisty.bean.User";
        String filedName = "age";

        Class<?> cls = Class.forName(className);
        Constructor<?> constructor = cls.getDeclaredConstructor();
        String setMethodName = "set" + filedName.toUpperCase().charAt(0) + filedName.substring(1);
        Field field = cls.getDeclaredField(filedName);
        Method method = cls.getDeclaredMethod(setMethodName, field.getType());

        User user = (User)constructor.newInstance();
        method.invoke(user, 20);
        System.out.println(user);
    }
}
```



## 手写Spring

Spring IoC容器实现原理：工厂模式 + 解析XML + 反射机制



### 准备工作

jar包管理：引入dom4j配置

```html
<dependency>
    <groupId>org.dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>2.1.4</version> <!-- 使用最新版本 -->
</dependency>
```

```html
<dependency>
    <groupId>jaxen</groupId>
    <artifactId>jaxen</artifactId>
    <version>1.2.0</version>
</dependency>
```

#### 使用者准备

配置文件

```html
<?xml version="1.0" encoding="UTF-8"?>
<beans>
    <bean id="user" class="com.myspring.bean.User">
        <property name="name" value="张三"/>
        <property name="age" value="18"/>
    </bean>

    <bean id="userDaoBean" class="com.myspring.bean.UserDao"/>

    <bean id="userService" class="com.myspring.bean.UserService">
        <property name="userDao" ref="userDaoBean"/>
    </bean>
</beans>
```

User对象

```java
package com.myspring.bean;

public class User {
    private String name;
    private int age;

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

UserDao对象

```java
package com.myspring.bean;

public class UserDao {
    public void insert() {
        System.out.println("mysql 保存用户信息");
    }
}
```

UserService对象

```java
package com.myspring.bean;

public class UserService {
    private UserDao userDao;

    public void save() {
        userDao.insert();
    }

    public UserDao getUserDao() {
        return userDao;
    }

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
}
```



#### 开发者框架编写

实现思路：实现ClassPathXmlApplicationContext类，私有化一个Map集合，存放bean对象的id，以及bean对象本身，提供一个getBean方法，从Map中取出对象返回

ApplicationContext 接口

```java
package org.myspringframework.core;

public interface ApplicationContext {
    Object getBean(String beanName);
}
```

ClassPathXmlApplicationContext 接口实现类的编写

```java
package org.myspringframework.core;

public class ClassPathXmlApplicationContext implements ApplicationContext {
    private Map<String, Object> singletonObject = new HashMap<>();

    public ClassPathXmlApplicationContext(String configLocations) {
        //解析配置文件，初始化所有Bean对象
        try {
            SAXReader reader = new SAXReader();
            InputStream in = this.getClass().getResourceAsStream("/" + configLocations);
            //读文件,获取标签
            Document document = reader.read(in);
            List<Node> nodes = document.selectNodes("//bean");
            //遍历bean标签，创建对象完成曝光
            for (Node node : nodes) {
                //向下转型，为使用Element接口方法
                Element beanElt = (Element) node;
                String id = beanElt.attributeValue("id");
                String className = beanElt.attributeValue("class");
                System.out.println("创建 " + id + " " + className);

                try {
                    Class<?> cls = Class.forName(className);
                    Constructor<?> declaredConstructor = cls.getDeclaredConstructor();
                    Object bean = declaredConstructor.newInstance();
                    singletonObject.put(id, bean);//曝光Bean对象，放入集合
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            }

            //遍历标签，完成赋值
            for (Node node : nodes) {
                try {
                    //向下转型，为使用Element接口方法
                    Element beanElt = (Element) node;
                    String id = beanElt.attributeValue("id");
                    String className = beanElt.attributeValue("class");
                    Class<?> clazz = Class.forName(className);
                    System.out.println("赋值 " + id + " " + className);
                    List<Element> properties = beanElt.elements("property");
                    for (Element property : properties) {
                        String propertyName = property.attributeValue("name");
                        System.out.println(propertyName);
                        String setMethodName = "set" + propertyName.substring(0, 1).toUpperCase() + propertyName.substring(1);
                        Field filed = clazz.getDeclaredField(propertyName);
                        Method setMethod = clazz.getDeclaredMethod(setMethodName, filed.getType());
                        String value = property.attributeValue("value");
                        String ref = property.attributeValue("ref");
                        if (value != null) {
                            //获取到的value值是字符串，需要将其转为对应类型的数据再放入
                            //支持byte short int long float double boolean char
                            //并支持对应的包装类型
                            String propertyTypeSimpleName = filed.getType().getSimpleName();
                            switch (propertyTypeSimpleName) {
                                case "int":
                                    setMethod.invoke(singletonObject.get(id), Integer.parseInt(value));
                                    break;
                                case "String":
                                    setMethod.invoke(singletonObject.get(id), value);
                                    break;
                                default:
                            }
                            //setMethod.invoke(singletonObject.get(id), "具体值");
                        }
                        if (ref != null) {
                            setMethod.invoke(singletonObject.get(id), singletonObject.get(ref));
                        }
                    }
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            }
        } catch (DocumentException e) {
            e.printStackTrace();
        }
    }

    @Override
    public Object getBean(String beanName) {
        return singletonObject.get(beanName);
    }
}
```

测试程序：使用自己编写的Spring框架进行测试

```java
package com.test;

import com.myspring.bean.UserService;
import org.junit.jupiter.api.Test;
import org.myspringframework.core.ApplicationContext;
import org.myspringframework.core.ClassPathXmlApplicationContext;

public class TestBean {
    @Test
    public void test() {
        ApplicationContext context = new ClassPathXmlApplicationContext("myspring.xml");
        Object user = context.getBean("user");
        System.out.println(user);
        UserService userService = (UserService)context.getBean("userService");
        userService.save();
        System.out.println(userService);
    }
}
```
