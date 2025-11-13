---
title: Kafka消息队列笔记（）
published: 2025-11-13
updated: 2025-11-13
description: ''
image: ''
tags: [MQ]
category: ''
draft: false 
---

# Kafka笔记

Kafka消息队列应用场景：异步处理、系统解耦、流量削峰、日志处理

交互模型：请求响应模型、生产者消费者模型

消息队列两种模式：点对点模式、发布订阅模型



Producers：将消息放入集群中

Consumers：将消息数据从集群中拉出来

Connectors：连接器可以将数据库中的数据导入Kafka，也可以将数据导出到数据库中

Stream Processors：流处理器可以从Kafka中拉取数据，也可以将数据写入Kafka

