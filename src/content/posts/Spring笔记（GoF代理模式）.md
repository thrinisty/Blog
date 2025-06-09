---
title: Spring笔记（GoF代理模式）
published: 2025-06-02
updated: 2025-06-02
description: '静态代理，动态代理'
image: ''
tags: [Spring]
category: 'Frame'
draft: false 
---

# Spring笔记

## 代理模式

### 定义作用

代理模式，是GoF的23种设计模式之一，也是AoP面向切面编程中的核心

:::note

作用：

当一个对象需要受保护的时候，可以使用代理对象完成某个行为

当需要完成功能增强的时候，使用代理对象增强功能

A对象和B对象无法直接交互的时候，可以使用代理模式解决

:::

为其他对象提供一种代理控制对这个对象的访问，在某些情况下，客户不想或者不能直接引用一个对象，可以通过一个称为“代理”的第三者实现间接引用，代理对象可以在客户端和目标对象之间起到中介的作用，并且可以通过代理对象增添功能或者过滤内容。

通过引入一个新的对象来实现对真实对象的操作或者将新的的对象作为真实对象的一个替身，这种实现机制为代理模式，而通过引入代理对象来间接访问另一个对象，这是代理模式的模式动机。



### 三个角色

:::tip

代理模式种拥有三大角色

1. 目标角色
2. 代理对象
3. 目标对象和代理对象的公共接口（前两者需要具有相同的行为）

:::



## 静态代理实现

有一个接口以及接口方法的实现类

```java
public interface OrderService {
    void generate();
    void modify();
    void detail();
}
```

```java
public class OrderServiceImpl implements OrderService {
    @Override
    public void generate() {
        System.out.println("订单生成");
    }

    @Override
    public void modify() {
        System.out.println("订单修改");
    }

    @Override
    public void detail() {
        System.out.println("订单详情");
    }
}
```

我们现在需要增添功能：对每个业务接口中业务方法的解决耗时，可以在每个业务的前后取时间相减，但是违背了OCP原则，且没有复用代码



我们使用代理模式处理，写一个OrderService的代理对象

```java
public class OrderServiceProxy implements OrderService{
    //写入公共接口，耦合度低
    private OrderService orderService;

    public OrderServiceProxy(OrderService orderService) {
        this.orderService = orderService;
    }

    @Override
    public void generate() {
        long begin = System.currentTimeMillis();
        orderService.generate();
        long end = System.currentTimeMillis();
        System.out.println("Generate Order Time: " + (end - begin));
    }

    @Override
    public void modify() {
        long begin = System.currentTimeMillis();
        orderService.modify();
        long end = System.currentTimeMillis();
        System.out.println("Modify Order Time: " + (end - begin));
    }

    @Override
    public void detail() {
        long begin = System.currentTimeMillis();
        orderService.detail();
        long end = System.currentTimeMillis();
        System.out.println("Detail Order Time: " + (end - begin));
    }
}
```

在主函数中调用代理对象的方法即可完成功能的增强

```java
public class Main {
    public static void main(String[] args) {
        OrderService orderService = new OrderServiceImpl();
        OrderService proxy = new OrderServiceProxy(orderService);
        proxy.generate();
        proxy.modify();
        proxy.detail();
    }
}
```

```
订单生成
Generate Order Time: 0
订单修改
Modify Order Time: 0
订单详情
Detail Order Time: 0
```

虽然解决了OCP的开闭问题，但是每个接口需要实现一个代理类，会导致类的数量急剧增多，我们在实际会使用到动态代理



## 动态代理

帮助我们在内存中动态的生成代理类（字节码），达到减少类的数量，提高复用性

JDK动态代理技术：只能代理接口

CGLIB：开源项目，高性能Code生成类库，底层通过继承的方式实现，可以代理类

Javassist动态代理技术：分析编辑创建Java字节码类库，通过使用Javassist对字节码作为JBoss实现动态“AOP”框架



### JDK动态代理

```java
package com.proxy.service;

public interface OrderService {
    void generate();
    void modify();
    void detail();
}
```

```java
package com.proxy.service;

public class OrderServiceImpl implements OrderService {
    @Override
    public void generate() {
        System.out.println("订单生成");
    }

    @Override
    public void modify() {
        System.out.println("订单修改");
    }

    @Override
    public void detail() {
        System.out.println("订单详情");
    }
}
```

我们通过一个Proxy对象传入三个参数 

1.类加载器 2.代理类实现的接口 3.调用处理器

```java
package com.proxy.test;

public class ProxyTest {
    public static void main(String[] args) {
        //创建目标对象
        OrderService target = new OrderServiceImpl();
        //创建代理类，借助jdk中的类
        //1.类加载器 2.代理类实现的接口 3.调用处理器
        Object proxyObj = Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                //为了创建代理类，需要获取目标类的类加载器
                target.getClass().getInterfaces(),
                //为了实现接口方法，需要获取类的所有接口对象
                new TimerInvocationHandler(target)
                //传入实现了处理接口的实例对象
                );
        OrderService proxy = (OrderService) proxyObj;
        //调用代理类
        proxy.generate();
        proxy.modify();
        proxy.detail();
    }
}
```

我们需要实现一个InvocationHandler接口的实现类，其中有一个invoke方法可供重写，这个方法在代理对象方法被调用的时候被调用，额外的，我们需要一个带参数的构造器，用以传入目标对象，这样在invoke中就可以指定目标对象，调用方法

```java
public class TimerInvocationHandler implements InvocationHandler {
    private Object target;

    public TimerInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        long begin = System.currentTimeMillis();
        Object object = method.invoke(target, args);
        long end = System.currentTimeMillis();
        System.out.println("Time: " + (end - begin));
        return object;//返回方法调用的返回值
    }
}
```

注意这里如果代理的目标方法存在返回值，需要在实现调用处理器的时候再次返回调用方法的返回值

这里因为创建代理类的时候实际上我们使用到的参数也就只有一个target目标对象，我们将创建代理对象的方法进行封装

```java
public class ProxyUtil {
    public static Object newProxyInstance(Object target) {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                //为了创建代理类，需要获取目标类的类加载器
                target.getClass().getInterfaces(),
                //为了实现接口方法，需要获取类的所有接口对象
                new TimerInvocationHandler(target)
                //传入实现了处理接口的实例对象
        );
    }
}
```

直接传入target对象即可获取到代理类

```java
Object proxyObj = ProxyUtil.newProxyInstance(target);
OrderService proxy = (OrderService) proxyObj;
```



### CGLIB动态代理

CGLIB 可以根据继承来代理，实现子类增强方法，调用父类的方法，注意不可以类不可以用final修饰父类，需要引入对应依赖，这里不再多做介绍
