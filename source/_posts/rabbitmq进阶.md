---
title: rabbitmq进阶
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-03 21:32:53
password:
summary:
tags:
- rabbitmq
categories:
- rabbitmq
---

## 消息传递

**mandatory**

`mandatory=true`，如果交换器无法根据自身的类型和路由键找到一个符合条件的队列，RabbitMQ会调用 `Basic.Return `命令将消息返回给生产者，生产者通过调用 `channel.addReturnListener `添加监听器接收返回结果
`mandatory=false`，上述情形下，RabbitMQ 将消息直接丢弃

**immediate**

`immediate=true`，如果交换器在消息路由到队列时发现队列上并不存在任何消费者，该消息将不会存入队列中。当与路由键匹配的所有队列都没有消费者时，该消息会通过 `Basic.Return` 返回生产者
**和mandatory相比，mandatory如果路由不到队列则返回消息，immediate如果队列中没有消费者则返回消息**

**备份交换器（Alternate Exchange）**

AE可以将未被路由的消息存储到 RabbitMQ 中。简化了`mandatory`+`addReturnListener `的编程逻辑。

```java
Map<String,Object> args = new HashMap<String,Object>();
args.put("alternate-exchange","myAe");

// 声明普通交换器（AE交换器作为备份交换器）
channel.exchangeDeclare("normalExchange","direct",true,false,args);
// 声明AE交换器
channel.exchangeDeclare("myAe","fanout",true,false,null);

// 普通队列 绑定 普通交换器
channel.queueBind("normalQueue","normalExchange","normalKey");

// 声明 未路由队列
channel.queueDeclare("unroutedQueue",true,false,false,null);
// 未路由队列 绑定 AE交换器
channel.queueBind("unroutedQueue","myAe","");
```

特殊情况

- 若备份交换器不存在，客户端和 RabbitMQ 服务端都不会有异常出现，消息丢失
- 若备份交换器没有绑定任何队列，客户端和 RabbitMQ 服务端都不会有异常出现，消息丢失
- 若备份交换器没有匹配任何队列，客户端和 RabbitMQ 服务端都不会有异常出现，消息丢失
- 若备份交换器和mandatory参数一起使用，该参数无效

## 过期时间（TTL）

**通过队列属性设置消息TTL**

```java
Map<String,Object> args = new HashMap<String,Object>();
args.put("x-message-ttl",6000); // 单位毫秒
channel.queueDeclare(queueName,durable,exclusive,autoDelete,args);
```

- 不设置TTL：该消息不会过期
- TTL为0：若直接可以投递到消费者，否则立刻被丢弃

消息过期：一旦过期，从队列中抹去。因为消息在队列头部，RabbitMQ只需要定期从头部开始扫描是否有过期消息即可。

**设置每条消息TTL**

在 `channel.basicPublish` 方法中加入 expiration 参数，单位毫秒

```java
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
builder.deliveryMode(2);
builder.expiration("60000");
AMQP.BasicProperties properties = builder.build();
channel.basicPublish(exchangeName,routingKey,mandatory,properties,"ttlTestMessage".getBytes());
```

消息过期：消息过期后，不会马上从队列中抹去，在即将投递到消费者之前判定。每条消息过期时间不同，删除所有过期消息势必要扫描整个队列，因此不如等到消息需要消费时再判定是否过期，若过期则删除。

**设置队列的TTL**

通过 `channel.queueDeclare` 方法中的 x-expires 参数可以控制队列被自动删除前处于未使用状态的时间

RabbitMQ 会确保在过期时间到达后将队列删除，在 RabbitMQ 重启后，过期时间会重置

## 死信队列

当消息在一个队列中变成死信，会被重新发送到死信交换器（Dead-Letter-Exchange, DLX），绑定DLX的队列称为死信队列

死信原因：1.消息被拒绝； 2.消息过期； 3. 队列达到最大长度

绑定死信队列：在 channel.queueDeclare 方法中设置 x-dead-letter-exchange 参数为此队列添加 DLX

## 延迟队列

消息当被发送以后，并不想让消费者立刻拿到消息，而是等待特定时间后，才能拿到消费

**场景：**

订单超时支付，延时队列做异常处理；

智能设备在指定时间进行工作，延时队列做指令推送；

**用法：**

- 每条消息设置为10秒过期时间
- 通过 exchange.normal 交换器把发送的消息存储到 queue.normal 队列中
- 消费者订阅 queue.dlx 队列
- 10秒后，消息过期转存到 queue.dlx ，消费者消费到了延迟10秒的这条消息

## 优先级队列

具有高优先级的队列有高的优先权，优先级高的消息优先被消费

```java
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
builder.priority(5);
AMQP.BasicProperties properties = builder.build();
channel.basicPublish("exchange_priority","rk_priority",properties,("message").getBytes());
```

- 默认优先级为0，最高为队列设置的最大优先级
- 如果Broker中有消息堆积，优先级高的消息可以被优先消费

## RPC实现

远程过程调用（Remote Procedure Call）,通过网络从远程计算上请求服务。应用部署在A服务器上，想要调用B服务器上提供的函数或者方法，需要通过网络表达调用的语义和传达调用的数据。

RPC的协议包括：Java RMI、WebService的RPC、THrift、RestfulAPI等。

```java
String callbackQueueName = channel.queueDeclare().getQueue();
BasicProperties props = new BasicProperties.Builder().replyTo(callbackQueueName).build();
channel.basicPublish("","rpc_queue",props,message.getBytes());
```

RPC 处理流程：

- 客户端启动时，创建一个匿名的回调队列
- 客户端为 RPC 请求设置2个属性：replyTo 告知 RPC 服务端回复请求时的目的队列；correlationId 标记一个请求
- 请求被发送到 rpc_queue 队列中
- RPC 服务端监听 rpc_queue 队列中的请求，当请求到来时，服务端会处理并且把带有结果的消息发送给客户端。接收队列是 replyTo 设定的回调队列''; 
- 客户端监听回调队列，有消息时，检查 correlationId 属性，如果与请求匹配，就是返回结果

## 持久化

持久化可以提高 RabbitMQ 的可靠性，防止在异常情况（重启、关闭、宕机）下数据丢失

持久化的各种情况

RabbitMQ 持久化分为3个部分：交换器的持久化、队列的持久化、消息的持久化

- 若交换器不设置持久化，服务重启后，交换器元数据丢失，但消息存在，不能将消息发送到该交换器中。长期使用的交换器，建议持久化。
- 若队列不设置持久化，服务重启后，队列元数据丢失，消息也会丢失；若队列设置持久化，消息不设置，服务重启后，队列元数据存在，但消息丢失
- 若所有消息设置持久化，会严重RabbitMQ性能，需要在吞吐量和可靠性之间做权衡
- 生产环境会设置镜像队列保证系统的高可用性
  

## 生产者确认

默认情况下，生产者不知道消息有没有正确到达服务器。因此引入事务和发送方确认机制。

**事务机制**

事务方法：

- channel.txSelect： 用于将当前信道设置成事务模式
- channel.txCommit：用于提交事务
- channel.txRollback：用于事务回滚

事务流程：

- 客户端发送 Tx.Select，将信道置为事务模式
- Broker回复 Tx.Select-Ok，确认已将信道置为事务模式
- Basic.Publish发完消息后，客户端发送 Tx.Commit 提交事务
- Broker 回复 Tx.Commit，确认事务提交

事务问题：事务机制会耗尽 RabbitMQ 的性能

**发送方确认机制**

1. 生产者将信道设置成 confirm 模式
2. 信道进入 confirm 模式后，所有在信道上发布的消息都会被指派一个唯一ID
3. 消息被投递到所有匹配的队列后，RabbitMQ 发送一个确认给生产者

发送方确认机制好处：相比于事务，它是异步非阻塞的。可以在等待信道返回确认的同时，继续发送下一条消息，当消息得到确认后，生产者可以通过回调方法处理确认消息。

发送方确认机制的优势在于不一定需要同步确认：

- 批量确认：每发送一批消息后，调用channel.waitForCOnfirms方法，等待服务器的确认返回
- 异步confirm方法：提供一个回调方法，服务器确认一条或者多条消息后客户端会回调这个方法进行处理。

注意：批量确认提升了confirm效率，但是返回Basic.Nack或者超时，客户端需要将这一个批次的消息全部重发，会带来明显的重复消息数量。消息经常丢失时，批量confirm性能应该不升反降。

## 消费端要点

**消息分发**

- 当 RabbitMQ 队列有多个消费者，消息会以轮询方式分发给消费者
- 但是这样会造成因为各机器性能不同而引起负载不均
- 消费端通过调用 channel.basicQos 方法，设置允许限制信道上的消费者保持最大未确认消息数量
- 一旦达到未确认消息数量上限，则停止向这个消费者发送消息，实现了“滑动窗口”效果
- Basic.Qos的使用对于拉模式消费方式无效

**消息顺序性**

顺序性指的是消费者消费的消息和发布者发布的消息的顺序是一致的

顺序性打破的情况：

-  生产者使用了事务机制，事务回滚后，补发信息可能在其它线程实现
- 启用 publiser confirm时，发生发生超时、中断，导致错序
- 生产者设置了延迟队列，但是超时时间设置的不一样
- 消息设置了优先级，消费端收到的消息必然不是顺序性的

**弃用 QueueingConsumer**

- 队列中有大量消息，可能导致内存溢出或假死，可以使用 Basic.Qos 方法得到有效解决
- QueueingConsumer会拖累一个 Connection 下的所有信道，使性能降低
- 同步调用 QueueingConsumer 会产生死锁

## 消息传输保障

**消息传输保障等级**

At most once：最多一次。消息可能丢失，但绝不会重复传输
At least once：最少一次。消息绝不会丢失，但可能重复传输
Exactly once：恰好一次。每条消息肯定会，有且传输一次
最少一次：需要考虑 事务、mandatory、持久化处理、autoAck
最多一次：无须考虑以上问题，随便发送与接收
恰好一次：RabbitMQ 目前无法保障。比如消费完Ack闪断，或者生产者发送消息到RabbitMQ，返回确认消息时网络闪断。

去重一般是通过业务客户端引入GUID实现