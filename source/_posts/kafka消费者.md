---
title: kafka消费者
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-23 21:07:18
password:
summary:
tags:
- kafka
categories:
- kafka
---

## 消费者组

Consumer Group 是 Kafka 提供的可扩展且具有容错性的消费者机制。

- Consumer Group 下可以有一个或多个 Consumer 实例。
- Group ID 是一个字符串，在一个 Kafka 集群中，它标识唯一的一个 Consumer Group。
- Consumer Group 下所有实例订阅的主题的单个分区，只能分配给组内的某个 Consumer 实例消费。这个分区当然也可以被其他的 Group 消费。
- 理想情况下，Consumer 实例的数量应该等于该 Group 订阅主题的分区总数

### 重平衡

Rebalance 本质上是一种协议，规定了一个 Consumer Group 下的所有 Consumer 如何达成一致，来分配订阅 Topic 的每个分区。

协调者(Coordinator)专门为 Consumer Group 服务，负责为 Group 执行 **Rebalance 以及提供位移管理和组成员管理**等。

所有 Broker 都会在启动时，创建和开启各自的 Coordinator 组件。

**Consumer Group 确定 Coordinator 所在的 Broker** 

第 1 步：确定由位移主题的哪个分区来保存该 Group 数据：`partitionId=Math.abs(groupId.hashCode() % offsetsTopicPartitionCount)`。

第 2 步：找出该分区 Leader 副本所在的 Broker，该 Broker 即为对应的 Coordinator。

注： Java Consumer API，能够自动发现并连接正确的 Coordinator。

**Rebalance 触发条件**

- 组成员数发生变更。比如有新的 Consumer 实例加入组或者离开组，抑或是有 Consumer 实例崩溃被“踢出”组。
- 订阅主题数发生变更。
- 订阅主题的分区数发生变更。

**Rebalance影响**

1、stop the world，所有 Consumer 实例都会停止消费，等待 Rebalance 完成

2、Rebalance 效率不高，需要重新分配所有分区

3、Rebalance很慢

**避免 Rebalance**

主要针对组成员数发生减少的情况。

Consumer 实例都会定期地向 Coordinator 发送心跳请求，表明它还存活着。session.timeout.ms + heartbeat.interval.ms

Consumer 端应用程序两次调用 poll 方法的最大时间间隔。默认值是 5 分钟，Consumer 程序如果在 5 分钟之内无法消费完 poll 方法返回的消息，那么 Consumer 会主动发起“离开组”的请求，Coordinator 也会开启新一轮 Rebalance。max.poll.interval.ms

### 位移主题

当 Kafka 集群中的第一个 Consumer 程序启动时，Kafka 会自动创建位移主题。自动创建的位移主题分区数是offsets.topic.num.partitions 50，副本数是offsets.topic.replication.factor 3。

1、将 Consumer 的位移数据作为一条普通的 Kafka 消息，保存到内部主题 _\_consumer_offsets 中。

位移主题消息的 Key 中格式：<Group ID，主题名，分区号 >，消息体保存了**位移值**和位移提交的元数据，诸如时间戳和用户自定义的数据等。

2、保存 Consumer Group 相关信息的消息

3、用于删除 Group 过期位移甚至是删除 Group 的消息。tombstone 消息，即墓碑消息

**提交位移**

自动提交位移和手动提交位移。

enable.auto.commit + auto.commit.interval.ms 控制。

自动提交位移省事，你不用操心位移提交，就能保证**消息消费不会丢失**。但是不可控。只要 Consumer 一直启动着，它就会无限期地向位移主题写入消息。

手动提交位移，consumer.commitSync 。

**删除位移主题中的过期消息**

kafka 使用 Compact 策略来删除位移主题中的过期消息。

Kafka 提供了专门的后台线程(Log Cleaner)定期地巡检待 Compact 的主题，看看是否存在满足条件的可删除数据。

对于同一个 Key 的两条消息 M1 和 M2，如果 M1 的发送时间早于 M2，那么 M1 就是过期消息。Compact 的过程就是扫描日志的所有消息，剔除那些过期的消息，然后把剩下的消息整理在一起。