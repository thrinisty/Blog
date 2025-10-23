---
title: RabbitMQ笔记（进阶使用）
published: 2025-10-21
updated: 2025-10-21
description: '消费端限流，超时设置，死信，延迟队列，事务消息，惰性队列，优先级队列群'
image: ''
tags: [MQ]
category: ''
draft: false 
---

# RabbitMQ笔记

## 进阶使用

### 消费端限流

设置prefetch进行限流，从消息队列中取消息每秒取一个

```yaml
spring:
  application:
    name: MQ
  rabbitmq:
    host: 115.190.182.149
    port: 5672
    username: admin
    password: 123456
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual
        prefetch: 1 #每秒从队列中取回消息的数量
```

### 消息超时

一旦设定一个过期时间，超过这个时间没有被取走的消息就会被删除

1.可以给队列指定过期时间，队列中所有消息就会按照对应的时间删除

在创建的时候指定即可

2.也可以给消息本身进行过期时间，如果两个都限制那么以小的为准

在我们发送消息的时候指定TTL即可，通过传入处理的messagePostProcessor调用setExpiration方法设置过期时间20s

```java
@GetMapping("/send")
public void send() {
    MessagePostProcessor messagePostProcessor = message -> {
        message.getMessageProperties().setExpiration("20000");
        return message;
    };
    rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, "Hello Order!", messagePostProcessor);
}
```



### 死信

我们在上面超时后，消息就会变为死信，死信产生的情况大概有三种

1.拒绝：消费者拒绝接收消息，并且不把消息重新放回目标队列，requeue=false

2.溢出：后来的消息将前面的消息顶掉

3.超时：到达时间未被消费

处理方式：

1.丢弃：对于不重要的消息进行丢弃

2.入库：把死信放入数据库，后续处理

3.监听：把死信放入死信队列，设置专门的消费端监听死信队列，进行后续处理

我们常用的是第三种方式

需要我们设置死信队列和交换机，还需要建立正常交换机和正常队列消息，在正常队列指定死信交换机和死信的路由键，这样我们超时后的消息就会被转发到死信交换机，再经由死信交换机发送给死信队列



### 延迟队列

1.借助消息超时时间+死信队列

通过上述同样的方法设置过期时间，对于死信队列的消息进行消费即可完成

2.安装RabbitMQ插件x-delayed-message，最多可以延迟两天的时间

将交换机中指定x-delayed-massage，然后还需要设置工作类型

设置延迟时间，也是通过messagePostProcessor调用设置message进行设置

```java
@GetMapping("/send")
public void send() {
    MessagePostProcessor messagePostProcessor = message -> {
        message.getMessageProperties().setHead("x-delay", "20000");
        return message;
    };
    rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, "Hello Order!", messagePostProcessor);
}
```



### 事务消息

只在生产者，不能控制消费者

如果我们有这么一个要求：有几条消息需要统一提交到队列中，我们就可以把这些消息放入缓存中统一发送

实现：需要提供RabbitTransactionManager为Bean对象，另外需要设置rabbitTemplate.setChannelTransacted(true)

```java
@Configuration
public class AnotherConfig {
    @Bean
    public RabbitTransactionManager rabbitTransactionManager(CachingConnectionFactory connectionFactory) {
        return new RabbitTransactionManager(connectionFactory);
    }
    @Bean
    public RabbitTemplate rabbitTemplate(CachingConnectionFactory connectionFactory) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setChannelTransacted(true); // 启用通道事务
        return template;
    }
}
```

```java
@RestController
@Transactional
public class SendController {
//    public static final String QUEUE_NAME = "queue.order";
    public static final String EXCHANGE_NAME = "test.back";
    public static final String ROUTING_KEY = "test";

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/send")
    public void send() {
        MessagePostProcessor messagePostProcessor = message -> {
            message.getMessageProperties().setExpiration("20000");
            return message;
        };
        rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, "Hello Order!", messagePostProcessor);
        int i = 10 / 0;
        rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, "Hello Order!", messagePostProcessor);

    }
}
```

这样就不会发送前一条消息，因为回滚了，不进行缓存的提交



### 惰性队列

Durable代表队列是长久的，消息会被放在磁盘上持久化，Transient代表队列是临时的，关闭后队列会消失，消息也消失

在队列满了的时候或者关闭broker的时候就会被持久化到磁盘上，另外写入磁盘的时候，生产者存入会被临时阻塞

应用场景：支持非常长的队列，（消费者停机、生产高峰），直接将内容写入硬盘，在消费时再取



### 优先级队列

我们需要设置有些消息更早被处理，可以设置优先级，用正整数表示优先级范围1-255，官网建议在1-5之间设置优先级，优先级越高，占用CPU，内存资源越多

队列声明使可以指定x-max-priority最高的值，默认为0（这个时候优先级无效）

```java
@GetMapping("/send")
public void send() {
    MessagePostProcessor messagePostProcessor = message -> {
        message.getMessageProperties().setPriority(4);
        return message;
    };
    rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, "Hello Order!", messagePostProcessor);
}
```
