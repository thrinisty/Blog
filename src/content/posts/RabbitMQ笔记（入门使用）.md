---
title: RabbitMQ笔记（入门使用）
published: 2025-10-20
updated: 2025-10-20
description: ''
image: './photo/RabbitMQ.png'
tags: [MQ]
category: ''
draft: false 
---

# RabbitMQ笔记

## 消息队列

我们在常规的访问通信是同步的，需要等到B返回响应后才可继续A的操作，而通过消息队列可以异步的继续远程的服务调用，不影响服务A的返回

![311](../images/311.png)

RabbitMQ结构

![312](../images/312.png)



## 入门使用

使用Docker拉取镜像这里注意要下载的是带管理界面的management，默认的好像没有15672端口的管理界面

```
docker pull rabbitmq:management
```

启用Docker运行RabbitMQ

```
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  --hostname my-rabbitmq-host \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=123456 \
  rabbitmq:management
```

