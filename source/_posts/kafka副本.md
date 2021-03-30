---
title: kafka副本
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-28 17:45:20
password:
summary:
tags:
- kafka
categories:
- kafka
---

主题可划分成若干个分，每个分区配置有若干个副本。副本（Replica），本质是一个只能**追加写消息的提交日志**。

## 副本分类

副本分成两类：领导者副本（Leader Replica）和追随者副本（Follower Replica）。每个分区在创建时都要选举一个副本，称为领导者副本，其余的副本自动称为追随者副本。

- 所有的读写请求都必须发往领导者副本所在的 Broker，由该 Broker 负责处理。

- 追随者副本是不对外提供服务的。从领导者副本异步拉取消息，并写入到自己的提交日志中，从而实现与领导者副本的同步。
- 领导者副本所在的 Broker 宕机时，Kafka 依托于 ZooKeeper 提供的监控功能能够实时感知到，并立即开启新一轮的领导者选举，从追随者副本中选一个作为新的领导者。

**优势**

1.方便实现“Read-your-writes”：

2.方便实现单调读（Monotonic Reads）：在多次消费消息时，它不会看到某条消息一会儿存在一会儿不存在。

## In-sync Replicas（ISR）

与 Leader 同步的副本，包括 Leader 副本。

标准就是 Broker 端参数 replica.lag.time.max.ms 参数值。这个参数的含义是 Follower 副本能够落后 Leader 副本的最长时间间隔。

## Unclean 领导者选举

非同步副本落后 Leader 太多，并选择这些副本作为新 Leader。

Broker 端参数 unclean.leader.election.enable 控制是否允许 Unclean 领导者选举。