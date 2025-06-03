---
title: Spring笔记（GoF代理模式）
published: 2025-06-02
updated: 2025-06-02
description: 'GoF之代理模式'
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
