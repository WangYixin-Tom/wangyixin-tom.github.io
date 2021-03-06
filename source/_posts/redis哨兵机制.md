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

- 主库或从库对 PING 命令的响应超时了，哨兵会标记为“主观下线”。

- 需有quorum 个实例判断主库为“主观下线”，才能判定主库为“客观下线”

### 选主

- 筛选当前在线从库，且网络连接状况较好；

- 选择从库优先级最高的从库

- 选择从库复制进度最快的

- 选择从库 ID 号小的

### 通知

- 通知从库执行replicaof，与新主库同步
- 通知客户端，像新主库请求

**通知客户端**的实现方法

1、哨兵会把新主库的地址写入自己实例的pubsub中。客户端需要订阅这个pubsub，当这个pubsub有数据时，客户端就能感知到主库发生变更，同时可以拿到最新的主库地址。

2、客户端需要支持主动去获取最新主从的地址进行访问。

## 哨兵集群pub/sub

**连接关系的实现**

- 哨兵-哨兵：哨兵订阅主库上的“__sentinel__:hello”，实现哨兵连接信息的发布获取
- 哨兵-从库：哨兵给主库发送 INFO 命令，主库接受到后，返回从库列表。从而哨兵可以连接从库
- 哨兵-客户端：客户端订阅哨兵消息


**哨兵Leader竞选 （总从切换）**

1、拿到半数以上的赞成票；

2、拿到的票数同时还需要大于等于仲裁所需的赞成票数（quorum ）。