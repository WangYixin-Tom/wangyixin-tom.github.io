---
title: redis哨兵机制
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-25 21:16:39
password:
summary:
tags:
- redis
categories:
- redis
---

## 问题

主库故障的相关问题：

1、确定主库故障

2、选择新的主库

3、新主库信息通知

## 基本功能

### 监控

- 哨兵进程周期性地给所有的主从库发送 PING 命令，检测它们是否仍然在线运行。

- 主库或从库对 PING 命令的响应超时了，哨兵会标记为主观下线。

- 需有quorum 个实例判断主库为主观下线，才能判定主库为客观下线

### 选主

- 筛选当前在线从库，且网络连接状况较好；

- 选择从库优先级最高的从库

- 选择从库复制进度最快的

- 选择从库 ID 号小的

### 通知

- 通知从库执行replicaof，与新主库同步
- 通知客户端，向新主库请求

**通知客户端**的实现方法

1、哨兵会把新主库的地址写入自己实例的pubsub中。客户端需要订阅这个pubsub，当这个pubsub有数据时，客户端就能感知到主库发生变更，同时可以拿到最新的主库地址。

2、客户端需要支持主动去获取最新主从的地址进行访问。

## 基于 pub/sub 机制的哨兵集群

**连接关系的实现**

- 哨兵-哨兵：哨兵订阅主库上的`__sentinel__:hello`，实现哨兵连接信息的发布获取
- 哨兵-从库：哨兵给主库发送 INFO 命令，主库接受到后，返回从库列表。从而哨兵可以连接从库
- 哨兵-客户端：客户端订阅哨兵消息


**哨兵Leader竞选 （总从切换）**

1、拿到半数以上的赞成票；

2、拿到的票数同时还需要大于等于仲裁所需的赞成票数（quorum ）。

## 节点间的内部通信机制

redis cluster 节点间采用 gossip 协议进行通信。所有节点都持有一份元数据，不同的节点如果出现了元数据的变更，就不断将元数据发送给其它的节点，让其它节点也进行元数据的变更。

- 优点：元数据的更新比较分散，不是集中在一个地方，更新请求会陆陆续续打到所有节点上去更新，降低了压力；

- 缺点：元数据的更新有延时，可能导致集群中的一些操作会有一些滞后。

交换的信息：信息包括故障信息，节点的增加和删除，hash slot 信息等等。

gossip 协议包含多种消息：

- meet：某个节点发送 meet 给新加入的节点，让新节点加入集群中，然后新节点就会开始与其它节点进行通信。redis-trib.rb add-node其实内部就是发送了一个 gossip meet 消息给新加入的节点，通知那个节点去加入我们的集群。
- ping：每个节点都会频繁给其它节点发送 ping，其中包含自己的状态还有自己维护的集群元数据，互相通过 ping 交换元数据。
- pong：返回ping和meeet，包括自己的状态和其他信息，也用于信息广播和更新。
- fail：某个节点判断另一个节点fail之后，就发送fail给其他节点，通知其他节点说，某个节点宕机了。

每个节点每秒会执行 10 次 ping，每次会选择 5 个最久没有通信的其它节点。当然如果发现某个节点通信延时达到了 cluster_node_timeout / 2，那么立即发送 ping，避免数据交换延时过长，落后的时间太长了。