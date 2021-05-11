---
title: rabbitmq消息发布
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-26 22:13:06
password:
summary:
tags:
- rabbitmq
categories:
- rabbitmq
---

## 可靠投递

------------

### mandatory

当交换器无法路由消息，RabbitMQ将回发Basic.Return消息到发布者，同时回发完整消息。
Basic.Return是异步的，在消息发布后的任何时候都可能发生。
在rabbitpy库中，客户端自动接收Basic.Return，并触发异常

示例程序：发布失败
```python
import datetime
import rabbitpy

# Connect to the default URL of amqp://guest:guest@localhost:15672/%2F
with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        # Create the message to send
        body = 'server.cpu.utilization 25.5 1350884514'
        message = rabbitpy.Message(channel,
                                   body,
                                   {'content_type': 'text/plain',
                                    'timestamp': datetime.datetime.now(),
                                    'message_type': 'graphite metric'})

        # Publish the message to the exchange with the routing key
        # "server-metrics" and make sure it is routed to the exchange
        message.publish('chapter2-example', 'server-metrics', mandatory=True)
```

示例程序：异常捕获
```python
import datetime
import rabbitpy

# Connect to the default URL of amqp://guest:guest@localhost:15672/%2F
connection = rabbitpy.Connection()
try:
    with connection.channel() as channel:
        properties = {'content_type': 'text/plain',
                      'timestamp': datetime.datetime.now(),
                      'message_type': 'graphite metric'}
        body = 'server.cpu.utilization 25.5 1350884514'
        message = rabbitpy.Message(channel, body, properties)
        message.publish('chapter2-example',
                        'server-metrics',
                        mandatory=True)
except rabbitpy.exceptions.MessageReturnedException as error:
    print('Publish failure: %s' % error)
```

### 发布者确认

发布者发送给RabbitMQ的每条消息，服务器发送一个确认（Basic.Ack）或者否认响应（Basic.Nack）。

示例程序：发布者确认
```python
import rabbitpy


with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        exchange = rabbitpy.Exchange(channel, 'chapter4-example')
        exchange.declare()
        channel.enable_publisher_confirms()
        message = rabbitpy.Message(channel,
                                   'This is an important message',
                                   {'content_type': 'text/plain',
                                    'message_type': 'very important'})
        if message.publish('chapter4-example', 'important.message'):
            print('The message was confirmed')
```

rabbitpy中没有使用回调的方法，而是在确认收到之前会一直阻塞。

### 备用交换器

处理无法路由的消息。当不可路由的消息发布到已经定义了备用交换器的交换器中时，它将被路由到备用交换器。


示例程序：备用交换器
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        my_ae = rabbitpy.Exchange(channel, 'my-ae',
                                  exchange_type='fanout')
        my_ae.declare()
        args = {'alternate-exchange': my_ae.name}
        exchange = rabbitpy.Exchange(channel,
                                     'graphite',
                                     exchange_type='topic',
                                     arguments=args)
        exchange.declare()
        queue = rabbitpy.Queue(channel, 'unroutable-messages')
        queue.declare()
        if queue.bind(my_ae, '#'):
            print('Queue bound to alternate-exchange')
```

### 基于事务的批量处理

确保消息投递成功。


```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        tx = rabbitpy.Tx(channel)
        tx.select()
        message = rabbitpy.Message(channel,
                                   'This is an important message',
                                   {'content_type': 'text/plain',
                                    'delivery_mode': 2,
                                    'message_type': 'important'})
        message.publish('chapter4-example', 'important.message')
        try:
            if tx.commit():
                print('Transaction committed')
        except rabbitpy.exceptions.NoActiveTransactionError:
            print('Tried to commit without active transaction')
```

由于错误无法路由消息时，将及时返回Basic.Return。发布者应发送TX.RollbackRPC请求并等待来自代理服务器的TX.Rollback相应，然后继续后续的工作。

### HA队列

允许队列在多个服务器拥有冗余副本。
发布者发送消息到集群中的任何节点。RabbitMQ节点同步队列中消息的状态。发布的消息被放入队列，并存储在每台服务器上。

示例代码：HA队列声明
```python
import rabbitpy

connection = rabbitpy.Connection()
try:
    with connection.channel() as channel:
        queue = rabbitpy.Queue(channel,
                               'my-ha-queue',
                               arguments={'x-ha-policy': 'all'})
        if queue.declare():
            print('Queue declared')
except rabbitpy.exceptions.RemoteClosedChannelException as error:
    print('Queue declare failed: %s' % error)
```


示例代码：HA队列指定节点
```python
import rabbitpy

connection = rabbitpy.Connection()
try:
    with connection.channel() as channel:
        arguments = {'x-ha-policy': 'nodes',
                     'x-ha-nodes': ['rabbit@node1',
                                    'rabbit@node2',
                                    'rabbit@node3']}
        queue = rabbitpy.Queue(channel,
                               'my-2nd-ha-queue',
                               arguments=arguments)
        if queue.declare():
            print('Queue declared')
except rabbitpy.exceptions.RemoteClosedChannelException as error:
    print('Queue declare failed: %s' % error)
```

### 消息持久化

delivery-mode=1，保存到内存；delivery-mode=2，保存到磁盘。服务器重启后，消息仍在队列中（队列必须声明为durable）。

如果rabbitmq经常等待操作系统响应读写请求，消息吞吐量将大大降低。


## rabbitmq回推

发布者发布消息太快，RabbitMQ发送Channel.Flow RPC方法，阻塞发布者。发布者不能发送消息直到收到另一条Channel.Flow命令。

示例代码：检测连接状态
```python
import rabbitpy

connection = rabbitpy.Connection()
print('Channel is Blocked? %s' % connection.blocked)

```