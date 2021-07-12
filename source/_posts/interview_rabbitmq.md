---
title: rabbitmq面试
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-05 20:05:22
password:
summary:
tags:
- interview
categories:
- interview
---

## MQ的优缺点

### 优点

- 异步 - 不需要立即处理的消息可以之后慢慢处理。异步处理可以提高系统吞吐量。
- 解耦 - 各个系统间通过消息通信，不用关心其他系统的处理。
- 削锋 - 可以通过消息队列支撑突发访问压力；可以缓解短时间内的高并发请求，不会因为突发超负荷请求而完全崩溃。

### 缺点

**系统复杂度提高**：需要考虑很多问题，如一致性问题、如何保证消息不被重复消费、如何保证消息可靠性传输等。

## RabbitMQ选型

可以支撑高并发、高吞吐、性能比较高，有非常完善便捷的后台管理界面。

支持集群化、高可用部署、消息高可靠支持，功能较为完善。

缺点：基于erlang语言开发的，分析里面的源码有一定门槛，较难进行深层次的源码定制和改造。

Kafka的优势在于专为超高吞吐量的实时日志采集、实时数据同步、实时数据计算等场景来设计。

## 什么是RabbitMQ

RabbitMQ是一款开源的，Erlang编写的，基于AMQP协议的消息中间件，主要负责接收、存储、转发消息。

## rabbitmq 的使用场景

（1）服务间异步通信

（2）顺序消费

（3）定时任务

（4）请求削峰

## 消息队列中推和拉模式

### push模式

服务器主动发送到用户的客户端，服务端保存状态。适用于消息量不大、消费能力强要求实时性高的情况下。

**优点**

- 及时性好，服务器端及时向客户端推送更新的动态信息
- consumer离线时，面临数据堆积或者数据丢失，折中方案是设定一个超时时间，当 Consumer 宕机时间超过这个阈值时，则清理数据；但这个时间阈值也并不太容易确定。

**缺点**

- **推送速率难以适应消费速率**，当生产者往 Broker 发送消息的速率大于消费者消费消息的速率时，随着时间的增长消费者那边可能就“爆仓”了

### pull模式

客户端主动从服务端拉取数据，客户端保存拉取信息状态

**优点**

- **消费者可以根据自身的情况来发起拉取消息的请求**。

**缺点**

- 消息延迟，消费者去拉取消息，只能不断地拉取，但是又不能很频繁地请求，太频繁了就变成消费者在攻击 Broker 了。
- **消息忙请求**，忙请求就是比如消息隔了几个小时才有，那么在几个小时之内消费者的请求都是无效的，在做无用功。优化方式：消费者如果尝试拉取失败，不是直接 return，而是把连接挂在那里 wait，服务端如果有新的消息到来，把连接拉起，返回最新消息。

## RabbitMQ基本概念

**Producer：生产者**，投递消息的程序

**Consumer：消费者**，接受消息的程序

**Broker：服务节点**，消息队列服务器实体

**Virtual host：**虚拟broker，用于逻辑隔离，最上层消息的路由。可以理解为虚拟 broker 。其内部均含有独立的 queue、exchange 和 binding 等，拥有独立的权限系统。

**Queue：队列**，RabbitMQ的内部对象，用于存储信息。多个消费可以订阅同一个队列，但消息会被平均分摊，而不是每个消费者都会受到所有的消息。

**Exchange：交换器**，生产者将消息发送到交换器，交换器将消息路由到一个或者多个队列中

**RoutingKey：路由键**，生产者将消息发给交换器的时候会指定一个路由键，用来指定路由规则

**Binding：绑定**，RabbitMQ通过绑定将交换器与队列关联起来，绑定时会指定BindingKey

**Connection：连接**，生产者或消费者和Broker之间的一条TCP连接

**Channel：信道**，建立在Connection上的虚拟连接，每条AMQP指令都通过信道完成交换器类型

**channel、exchange 和 queue 这些是逻辑概念，还是对应着进程实体？**

- queue有自己的erlang进程;
- exchange保存binding关系的查找表;
- channel是实际进行路由工作的实体，根据routing_key将消息投递给queue。channel是在tcp连接上的虚拟链接，amqp命令通过channel发送，
  一个线程允许使用多个channel，多线程共享同一个socket。

## RabbitMQ的工作模式

**简单模式**：它包含一个生产者、一个消费者和一个队列。生产者向队列里发送消息，消费者从队列中获取消息并消费。

**工作模式**：向多个互相竞争的消费者发送消息的模式，它包含一个生产者、两个消费者和一个队列。两个消费者同时绑定到一个队列上去，当消费者获取消息处理耗时任务时，空闲的消费者从队列中获取并消费消息。

**发布/订阅模式**：同时向多个消费者发送消息的模式（类似广播的形式），它包含一个生产者、两个消费者、两个队列和一个交换机。两个消费者监听各自队列，两个队列绑定到交换机上去，生产者通过发送消息到交换机，所有消费者接收并消费消息。

**路由模式**：根据`路由键`选择性给多个消费者发送消息的模式，它包含一个生产者、两个消费者、两个队列和一个交换机。两个消费者监听各自队列，两个队列通过`路由键`绑定到交换机上去。生产者发送消息到交换机，交换机通过`路由键`转发到不同队列，队列绑定的消费者接收并消费消息。

**主题模式**：根据`路由键匹配规则`选择性给多个消费者发送消息的模式，它包含一个生产者、两个消费者、两个队列和一个交换机。两个消费者监听各自队列，两个队列通过`路由键匹配规则`绑定到交换机上去。生产者发送消息到交换机，交换机通过`路由键匹配规则`转发到不同队列，队列绑定的消费者接收并消费消息。

## 消息在什么时候会变成死信?

- 消息拒绝并且没有设置重新入队
- 消息过期
- 消息堆积，并且队列达到最大长度，先入队的消息会变成DL

## 如何保证RabbitMQ消息的顺序性？

1、需要保证顺序性的消息使用一个queue对应一个 consumer。

2、消息在被创建时，都将被赋予一个全局唯一的、单调递增的、连续的序列号，可以通过一个全局计数器来实现这一点。可以在消费端实现前一条消息未消费，不处理下一条消息；也可以在生产端实现前一条消息未处理完毕，不发布下一条消息。

## 如何保证消息不丢失？

1. 生产者发送消息至MQ的数据丢失

解决方法:在生产者端开启comfirm 确认模式，然后如果写入了 RabbitMQ 中，RabbitMQ 会给你回传一个 ack 消息，告诉你说这个消息 ok 了。

2. MQ收到消息，暂存内存中，还没消费，自己挂掉，数据会都丢失

解决方式：MQ设置为持久化。将内存数据持久化到磁盘中

3. 消费者刚拿到消息，还没处理，挂掉了，MQ又以为消费者处理完

解决方式：用 RabbitMQ 提供的 ack 机制，每次处理完的时候， ack 一把。

## 如何保证消息不被重复消费？

1、设置操作的幂等性。

2、生产者发送每条数据的时候，里面加一个全局唯一的 id，消费者通过全局唯一ID检查是否消费过。

3、基于数据库的唯一键来保证重复数据不会重复插入多条。

## **消息如何分发？**

若该队列至少有一个消费者订阅，消息将以循环（round-robin）的方式发送给消费者。每条消息只会分发给一个订阅的消费者。通过路由可实现多消费的功能

## 消息怎么路由？

消息将拥有一个路由键（routing key），在消息创建时设定。通过队列路由键，可以把队列绑定到交换器上。

消息到达交换器后，RabbitMQ 会将消息的路由键与队列的路由键进行匹配（针对不同的交换器有不同的路由规则）；

常用的交换器主要分为一下三种：

fanout：如果交换器收到消息，将会广播到所有绑定的队列上

direct：如果路由键完全匹配，消息就被投递到相应的队列

topic：可以使来自不同源头的消息能够到达同一个队列。 使用 topic 交换器时，可以使用通配符

## 消息基于什么传输？

由于 TCP 连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。

RabbitMQ 使用信道的方式来传输数据。信道是建立在**真实的 TCP 连接内的虚拟连接**，且每条 TCP 连接上的信道数量没有限制。

## 什么情况下会出现blackholed问题？

blackholed问题是指，向exchange投递了 message，而由于各种原因导 致该message丢失，但发送者却不知道。可导致blackholed的情况：

- 向未绑定 queue 的 exchange 发送 message；
- exchange 以 binding_key key_A 绑定了 queue queue_A，但向该 exchange 发送 message 使用的 routing_key却是 key_B。

## 如何防止出现blackholed问题？

如果在执行Basic.Publish时设置`mandatory=true`，则在遇到可能出现blackholed情况时，服务器会通过返回Basic.Return告之当前 message无法被正确投递。

## Basic.Reject的用法是什么？

该信令可用于consumer对收到的message进行reject。

若在该信令中设 置`requeue=true`，则当RabbitMQ server收到该拒绝信令后，会将该 message重新发送到下一个consumer处（理论上仍 可能将该消息发送给当前consumer）。

若设置`requeue=false`，则 RabbitMQ server在收到拒绝信令后，将直接将该message从queue中移除。


## 如何保证消息不被重复消费？

**重复消费场景：**因为网络传输等故障，消费确认信息没有传送到消息队列，导致消息队列不知道已经消费过该消息了，再次将消息分发给其他的消费者。

**解决思路**：保证消息的唯一性，就算是多次传输，不要让消息的多次消费带来影响；保证消息等幂性；

比如：在写入消息队列的数据做唯一标识，消费消息时，根据唯一标识判断是否消费过；

## 如何确保消息正确地发送至 RabbitMQ？ 如何确保消息接收方消费了消息？

**发送方确认模式**

将信道设置成 confirm 模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的 ID。

一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会**发送一个确认给生产者**（包含消息唯一 ID）。

如果 RabbitMQ 发生内部错误从而导致消息丢失，会发送一条 **nack**（notacknowledged，未确认）消息。

发送方确认模式是异步的，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息。

**接收方确认机制**

消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作）。只有消费者确认了消息，RabbitMQ 才能安全地把消息从队列中删除。

这里并没有用到超时机制，RabbitMQ 仅通过 Consumer 的连接中断来确认是否需要重新发送消息。也就是说，只要连接不中断，RabbitMQ 给了 Consumer 足够长的时间来处理消息。保证数据的最终一致性；

**特殊情况**

如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ 会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要去重）
如果消费者接收到消息却没有确认消息，连接也未断开，则 RabbitMQ 认为该消费者繁忙，将不会给该消费者分发更多的消息。

## 如何保证RabbitMQ消息的可靠传输？

丢失又分为：生产者丢失消息、消息列表丢失消息、消费者丢失消息；

**生产者丢失消息：**RabbitMQ提供transaction和confirm模式来确保生产者不丢消息；

transaction机制：发送消息前，开启事务,然后发送消息，如果发送过程中出现什么异常，事务就会回滚,如果发送成功则提交事务。然而，这种方式有个缺点：吞吐量下降；

confirm模式用的居多：一旦channel进入confirm模式，所有在该信道上发布的消息都将会被指派一个唯一的ID（从1开始），一旦消息被投递到所有匹配的队列之后；rabbitMQ就会发送一个ACK给生产者（包含消息的唯一ID），这就使得生产者知道消息已经正确到达目的队列了；如果rabbitMQ没能处理该消息，则会发送一个Nack消息给你，你可以进行重试操作。

**消息队列丢数据：消息持久化。**开启持久化磁盘的配置。

这个持久化配置可以和confirm机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个Ack信号。

队列持久化+消息持久化。

消费者在收到消息之后，处理消息之前，会自动回复RabbitMQ已收到消息；

如果这时处理消息失败，就会丢失该消息；

解决方案：处理消息成功后，**手动回复确认消息**。

## 为什么不应该对所有的 message 都使用持久化机制？

一般仅对关键消息作持久化处理，且应该保证关键消息的量不会导致性能瓶颈。否则会导致性能的下降，因为写磁盘比写 RAM 慢的多。

其次，message 的持久化机制用在 RabbitMQ 的内置 cluster 方案时会出现“坑爹”问题。

## 集群

### 如何保证高可用的？RabbitMQ 的集群

普通集群模式，在多台机器上启动多个 RabbitMQ 实例。 **queue，只会放在一个 RabbitMQ 实例上，但是每个实例都同步 queue 的元数据。**消费的时候，实际上如果连接到了另外一个实例，那么那个实例会从 queue 所在实例上拉取数据过来。这方案主要是提高吞吐量的，就是说让集群中多个节点来服务某个 queue 的读写操作。

**镜像集群模式**： RabbitMQ 的高可用模式。在镜像集群模式下，你创建的 queue，**无论元数据还是 queue 里的消息都会存在于多个实例上**，就是说，每个 RabbitMQ 节点都有这个 queue 的一个完整镜像，包含 queue 的全部数据的。然后每次你写消息到 queue 的时候，都会自动把消息同步到多个实例的 queue 上。

好处在于，任何一个机器宕机了，其它机器还包含了这个 queue 的完整数据，别的 consumer 都可以到其它节点上去消费数据。

坏处在于，这个性能开销大，消息需要同步到所有机器上，导致网络带宽压力和消耗很重。

### rabbitmq 对集群节点停止顺序有要求吗?

RabbitMQ 对集群的停止的顺序是有要求的，应该先关闭内存节点，最后再关闭磁盘节点.如果顺序恰好相反的话，可能会造成消息的丢失

### rabbitmq 集群有什么用?

- 高可用: 某个服务器出现问题，整个 RabbitMQ 还可以继续使用
- 高容量: 集群可以承载更多的消息量

### RAM node 和 disk node 的区别？

RAM node 仅将相关元数据保存到内存中，

disk node会在内存和磁盘中均进行存储。

要求在RabbitMQ cluster中至少存在一个disk node。

### **rabbitmq 每个节点是其他节点的完整拷贝吗？为什么？**

不是，原因有以下两个：

1. 存储空间的考虑：如果每个节点都拥有所有队列的完全拷贝，这样新增节点不但没有新增存储空间，反而增加了更多的冗余数据；
2. 性能的考虑：如果每条消息都需要完整拷贝到每一个集群节点，那新增节点并没有提升处理消息的能力，反而可能更糟。

### **rabbitmq 集群中唯一一个磁盘节点崩溃了会发生什么情况？**

如果唯一磁盘的磁盘节点崩溃了，不能进行以下操作：

不能创建队列

不能创建交换器

不能创建绑定

不能添加用户

不能更改权限

不能添加和删除集群节点

唯一磁盘节点崩溃了，集群是可以保持运行的，但你不能更改任何东西。

### 元数据

- 非 cluster 模式下，元数据主要分为 Queue 元数据（queue 名字和属性等）、Exchange 元数据（exchange 名字、类型和属性等）、Binding 元数据（存放路由关系的查找表）、Vhost 元数据（vhost 范围内针对前三者的名字空间约束和安全属性设置）。
- cluster 模式下，还包括 cluster 中 node 位置信息和 node 关系信息。元数据按照 erlang node 的类型确定是仅保存于 RAM 中，还是同时保存在 RAM 和 disk 上。元数据在cluster 中是全 node 分布的。