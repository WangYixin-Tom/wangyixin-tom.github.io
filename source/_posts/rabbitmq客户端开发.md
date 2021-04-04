---
title: rabbitmq客户端开发
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-03 20:58:05
password:
summary: rabbitmq客户端开发的说明。
tags:
- rabbitmq
categories:
- rabbitmq
---

## 连接 RabbitMQ

```java
ConnectionFactory factory = new ConnectionFactory();
factory.setUsername(USERNAME);
factory.setPassword(PASSWORD);
factory.setVirtualHost(virtualHost);
facotry.setHost(IP_ADRESS);
factory.setPort(PORT);
// 通过URI方式
// factory.setUri("amqp://userName;password@ipAddress:portNumber/virtualHost");
Connection conn = factory.newConnection();
Channel channel = conn.createChannel();
```

Connection 可以创建多个 Channel 实例，但是 Channel 实例不能在线程间共享，应用程序应该为每一个线程开辟一个 Channel

## 使用交换器和队列

```java
// 持久化的、非自动删除的、绑定类型为direct的交换器
channel.exchangeDeclare(exchangeName,"direct",true);
// 非持久化的、排他的、自动删除的队列、RabbitMQ自动生成队列名
String queueName = channel.queueDeclare().getQueue();
// 持久化的、非排他的、非自动删除的、确定的已知名称
// channel.queueDeclare(queueName,true,false,false,null);
channel.queueBind(queueName,exchangeName,routingKey);
```

exchangeDeclare定义：交换器名称、类型、是否持久化、是否自动删除、是否内置

queueDeclare定义：队列名称、是否持久化、是否排他、是否自动删除。排他队列仅对声明它的连接可见，并在连接断开时自动删除。

queueBind定义：队列名称、交换器名称、路由键

## 发送消息

```java
byte[] messageBodyBytes = "Hello,world".getBytes();
// 普通发送
channel.basicPublish(exchangeName,routingKey,null,messageBodyBytes);
// 控制发送,使用mandatory
channel.basicPublish(exchangeName,routingKey,mandatory,MessageProperties.PERSiSTENT_TEXT_PLAIN,messageBodyBytes);
```

## 消费消息

消费模式分为退模式和拉模式。推模式采用Basic.Consume,，拉模式采用Basic.Get

**推模式**

```java
boolean autoAck = false;
channel.basicQos(64);
channel.basicConsume(queueName,autoAck,"myConsumerTag",
    new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag,Envelope envelope,AMQP.BasicProperties properties,byte[] body) throws IOException{
            String routingKey = envelope.getRoutingKey();
            String contentType = properties.getContentType();
            long deliveryTag = envelope.getDeliveryTag();
            channel.basicAck(deliveryTag, false);
        }
    }
);
```

显示设置autoAck为false，接收消息后显示ack操作，可以防止消息不必要地丢失

每个Channel都拥有独立的线程，最常用做法是一个Channel对应一个消费者

如果一个Channel维持多个消费者，其中一个消费者一直运行，其它消费者callback会被耽搁

**拉模式**

```java
GetResponse response = channel.basicGet(QUEUE_NAME,false);
System.out.pringln(new String(response.getBody()));
channel.basicAck(response.getEnvelope().getDeliveryTag(),false);
```

如果设置autoAck为false，同样需要调用channel.basicAck来确认消息已被成功接收

如果是持续订阅，且需要高吞吐量推荐使用推模式。否则单条信息获取，可以使用拉模式。

## 消费端的确认与拒绝

**消息确认机制**

RabbitMQ 提供消息确认机制，消费者订阅队列时，需要制定autoAck参数
autoAck参数为false，RabbitMQ会等待消费者显式回复确认信号再从内存移去消息
autoAck参数为true，RabbitMQ会自动把发出的消息置为确认，然后删除
**拒绝消息**

消费者接收到消息后，可以通过 Basic.Reject 拒绝消息；批量拒绝消息可以使用Basic.Nack
如果requeue参数为true，服务端收到拒绝信号后，会重新把消息发送给下一个消费者
如果requeue参数为false，服务端收到拒绝信号后，会把消息从队列中移除
如果requeue参数为false，可以启用“死信队列”，分析消息追踪问题
**注意要点**

RabbitMQ 服务端中的队列分成了两个部分：等待投递的消息；已投递未确认的消息
如果 RabbitMQ 一直没有收到确认，并且消费者已经断开连接，会安排此消息重新进入队列，投递给下一个消费者
RabbitMQ 不会为未确认的消息设置过期时间，允许长时间消费。

## 关闭连接

```java
channel.close();
conn.close();
```

显式关闭 Channel 不是必须的，在Connection关闭的时候 Channel 也会自动关闭