---
title: kafka消息丢失
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-23 19:13:33
password:
summary:
tags:
- kafka
categories:
- kafka
---

kafka 只对“已提交”的消息（committed ）做有限度的持久化保证。

## 避免消息丢失

- 不要使用 `producer.send(msg)`，而要使用` producer.send(msg, callback)`。一定要使用带有回调通知的 send 方法。
- 设置 `acks = all`。`acks `是 Producer 的一个参数，代表了你对“已提交”消息的定义。如果设置成 all，则表明所有副本 Broker 都要接收到消息，该消息才算是“已提交”。这是最高等级的“已提交”定义。
- 设置 `retries` 为一个较大的值。当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了 `retries > 0` 的 Producer 能够自动重试消息发送，避免消息丢失。
- 设置 `unclean.leader.election.enable = false`。如果一个 Broker 落后原先的 Leader 太多，那么它一旦成为新的 Leader，必然会造成消息的丢失。故一般设置成 false。
- 设置 `replication.factor >= 3`。防止消息丢失的主要机制就是冗余，最好将消息多保存几份。
- 设置 `min.insync.replicas > 1`。消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。在实际环境中千万不要使用默认值 1。
- 确保 `replication.factor > min.insync.replicas`。如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。推荐设置成 `replication.factor = min.insync.replicas + 1`。
- 确保消息消费完成再提交。Consumer 端有个参数 `enable.auto.commit`，最好把它设置成 `false`，并采用手动提交位移的方式。对于单 Consumer 多线程处理的场景而言是至关重要的