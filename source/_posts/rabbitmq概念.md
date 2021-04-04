---
title: rabbitmq概念
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-03 18:50:50
password:
summary: rabbit相关概念介绍
tags:
- rabbitmq
categories:
- rabbitmq
---

Rabbitmq是生产者与消费者模型，负责接收、存储、转发消息。

![](moxin.png)

**Producer：生产者**

**消息**包含2部分：

- **消息体（payload）：**一般是一个带有业务逻辑结构的数据，比如一个JSON字符串。

- **标签（Label）：**用来描述这条消息，比如一个交换器的名称和一个路由键。

**Consumer：消费者**

当消费者消费一条消息时，只是消费消息的消息体（payload）。在消息路由的过程中，消息的标签会丢弃，存入到队列中的消息只有消息体，消费者也只会消费到消息体。

**Broker：服务节点**

**Queue：队列**，RabbitMQ的内部对象，用于存储信息。多个消费可以订阅同一个队列，但消息会被平均分摊，而不是每个消费者都会受到所有的消息。

**Exchange：交换器**，生产者将消息发送到交换器，交换器将消息路由到一个或者多个队列中

**RoutingKey：路由键**，生产者将消息发给交换器的时候会指定一个路由键，用来指定路由规则

**Binding：绑定**，RabbitMQ通过绑定将交换器与队列关联起来，绑定时会指定BindingKey

**Connection：连接**，生产者或消费者和Broker之间的一条TCP连接

**Channel：信道**，建立在Connection上的虚拟连接，每条AMQP指令都通过信道完成交换器类型

**交换器类型**

fanout：把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中

direct：把消息路由到BindingKey和RoutingKey完全匹配的队列中

topic：把消息路由到BindingKey和RoutingKey相匹配的队列中

headers：根据发送消息内容中的headers进行匹配，性能比较差
**运转流程**
**生产者发送消息**

- 生产者连接到 RabbitMQ Broker，建立一个连接（Connection），开启一个信道（Channel）

- 生产者声明一个交换器，并设置相关属性（交换机类型、持久化）

- 生产者声明一个队列，并设置相关属性（排他、持久化、自动删除）

- 生产者通过路由键将交换器和队列绑定起来

- 生产者发消息至RabbitMQ Broker（包含路由键、交换器信息）

- 相应的交换器根据接收到的路由键查找相匹配的队列

- 若找到队列，则把消息存入；若没有，则丢弃或者回退给生产者

- 关闭信道

- 关闭连接

**消费者接收消息**

- 消费者连接到 RabbitMQ Broker ，建立一个连接（Connection），开启一个信道（Channel）
- 消费者向 RabbitMQ Broker 请求消费对应队列中的消息
- 等待 RabbitMQ Broker 回应并投递相应队列中的消息，消费者接收消息
- 消费者确认（ack）接收到的消息
- RabbitMQ 从队列中删除相应已经被确认的消息
- 关闭信道
- 关闭连接

