---
title: rabbitmq消息消费
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-26 22:16:01
password:
summary:
tags:
- rabbitmq
categories:
- rabbitmq
---

## 消费方法

### **Basic.Get**

 - 每次接收消息必须发送一次请求 
 - 有消息可用，RabbitMQ返回Basic.GetOk以及消息
 - 无消息可用，RabbitMQ返回Basic.GetEmpty 应用程序需要评估RPC响应以及是否接收到消息。

示例程序
```
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        queue = rabbitpy.Queue(channel, 'test-messages')
        queue.declare()
        while True:
            message = queue.get()
            if message:
                message.pprint()
                # 确认消息
                message.ack()
                if message.body == 'stop':
                    break
```

### **Basic.Consume**

- 消费者可用时，异步方式发送消息
- 应用程序自动接收消息，直到Basic.Cancel
- 仍然需要确认消息

示例程序
```python
import rabbitpy

for message in rabbitpy.consume('amqp://guest:guest@localhost:5672/%2f',
                                'test-messages'):
    message.pprint(）
    # 消息确认
    message.ack()
```

**消费者标签**
应用程序发出Basic.Comsume时，创建唯一字符串（消费者标签），标识应用程序。RabbitMQ每次都会把该字符串和消息一同发送给应用程序。
客户端库对消费者标签封装，以确定如何处理消息。开发者不用处理消费者标签。

示例代码：监听消息直到，收到停止消息
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        for message in rabbitpy.Queue(channel, 'test-messages'):
            message.pprint()
            message.ack()
            if message.body == 'stop':
                break
```


### **对比**
Consume吞吐量更大。Get包含了每条消息的同步通信开销。

## 消费性能优化

### 1、no-ack

应用程序发送Basic.Comsume请求时，设置no-ack。表明消费者不进行消费确认。

示例代码：消费不确认
```
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        queue = rabbitpy.Queue(channel, 'test-messages')
        for message in queue.consume_messages(no_ack=True):
            message.pprint()
```

### 2、预取

QoS（Quality of service）中，可设置消费者预先接收一定数量的消息。Basic.Qos一般在Basic.Consume之前设置。

示例程序：指定QoS
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        #预取数为10
        channel.prefetch_count(10)
        for message in rabbitpy.Queue(channel, 'test-messages'):
            message.pprint()
            message.ack()
```

应用程序不需要确认每条消息，可确认所有以前未读消息。

示例程序：多消息确认
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        channel.prefetch_count(10)
        for message in rabbitpy.Queue(channel, 'test-messages'):
            message.pprint()
            unacknowledged += 1
            if unacknowledged == 10:
                # 确认所有未确认消息
                message.ack(all_previous=True)
                unacknowledged = 0

```

### 3、事务

事务允许消费者应用程序提交和回滚批量操作。不适用QoS时，可以获得轻微的性能提升。

## 拒绝消息

### Basic.Reject

通知rabbitmq无法处理投递的消息（拒绝一个消息），可指示rabbitMQ丢弃消息或使用requeue重发消息。

示例程序：消息拒绝
```
import rabbitpy

for message in rabbitpy.consume('amqp://guest:guest@localhost:5672/%2f',
                                'test-messages'):
    message.pprint()
    print('Redelivered: %s' % message.redelivered)
    message.reject(True)
```

### Basic.Nack

同时拒绝多个消息

### 死信交换器（DLX）

创建队列时声明该交换器用于保存被拒绝的消息。队列x-dead-letter-exchange参数(RPC请求)指定死信交换器。

示例程序：指定死信交换器
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        #死信交换器
        rabbitpy.Exchange(channel, 'rejected-messages').declare()
        queue = rabbitpy.Queue(channel, 'dlx-example',
                               dead_letter_exchange='rejected-messages')
        queue.declare()

```

## 控制队列

### 临时队列

**自动删除队列**
消费者完成连接和检索消息，所有消费者断开连接时，队列将被删除。

示例程序：自动删除队列auto_delete=True
```
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        queue = rabbitpy.Queue(channel, 'ad-example', auto_delete=True)
        queue.declare()
```

**只允许单个消费者**
只有单个消费者能够消费队列中的消息。消费者断开连接后，会自动删除队列。

示例程序：独占队列exclusive
```
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        queue = rabbitpy.Queue(channel, 'exclusive-example',
                               exclusive=True)
        queue.declare()
```

**自动过期队列**
如果一段时间没有使用该队列就删除它，一般用于RPC回复队列。

示例程序：自动过期队列
```
import rabbitpy
import time

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        queue = rabbitpy.Queue(channel, 'expiring-queue',
                               arguments={'x-expires': 1000})
        queue.declare()
        messages, consumers = queue.declare(passive=True)
        time.sleep(2)
        try:
            messages, consumers = queue.declare(passive=True)
        except rabbitpy.exceptions.AMQPNotFound:
            print('The queue no longer exists')
```

### 永久队列

**队列持久性**
服务器重启后队列仍然存在。
示例程序：持久队列

```
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        queue = rabbitpy.Queue(channel, 'durable-queue',
                               durable=True)
        if queue.declare():
            print('Queue declared')
```

**队列消息自动过期**
同时指定死信交换器和消息TTL，过期消息将成为死信消息。

示例程序：消息TTL
```
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        queue = rabbitpy.Queue(channel, 'expiring-msg-queue',
                               arguments={'x-message-ttl': 1000})
         queue.declare()

```

**最大队列长度**
一旦达到最大值，添加新消息时，删除队列前端的消息。

声明队列时，如果指定死信交换器，前端移除的消息将成为死信。

示例程序：最大长度队列
```
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        queue = rabbitpy.Queue(channel, 'max-length-queue',
                               arguments={'x-max-length': 1000})
        queue.declare()

```


### 队列设置参数

| 参数                      | 说明                                 |
| ------------------------- | ------------------------------------ |
| x-dead-letter-exchange    | 死信交换器，路由不重发且被拒绝的消息 |
| x-dead-letter-routing-key | 死信消息的可选路由键                 |
| x-expires                 | 队列在指定的毫秒数后删除             |
| x-ha-proxy                | 创建HA队列                           |
| x-ha-nodes                | HA队列分布的节点                     |
| x-max-length              | 队列的最大消息数                     |
| x-message-ttl             | 毫秒为单位的队列过期时间             |
| x-max-priority            | 队列优先级排序                       |