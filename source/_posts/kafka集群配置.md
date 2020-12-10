---
title: kafka集群配置
top: false
cover: false
toc: true
mathjax: true
date: 2020-12-10 21:12:03
password:
summary:
tags:
- kafka
categories:
- kafka
---

## Broker 端参数

- log.dirs：Broker 需要使用的若干个文件目录路径，必须指定；最好不同路径挂载到不同的物理磁盘，提升读写性能且能能够实现故障转移
- log.dir：单个路径

- zookeeper.connect：zookeeper端口
- listeners：访问kafka的监听器
- advertised.listeners：Broker 用于对外发布的监听器
- auto.create.topics.enable：是否允许自动创建 Topic，建议最好设置成 false
- unclean.leader.election.enable：是否允许 Unclean Leader 选举，是否能让那些落后太多的副本竞选 Leader,建议false
- auto.leader.rebalance.enable：是否允许定期进行 Leader 选举，成本太高，建议false
- log.retention.{hours|minutes|ms}：控制一条消息数据被保存多长时间。从优先级上来说 ms 设置最高、minutes 次之、hours 最低
- log.retention.bytes：这是指定 Broker 为消息保存的总磁盘容量大小。默认-1
- message.max.bytes：控制 Broker 能够接收的最大消息大小。

## Topic 级别参数

- retention.ms： Topic 消息被保存的时长。默认是 7 天。会覆盖掉 Broker 端的全局参数值。

- retention.bytes： Topic 预留多大的磁盘空间。通常在多租户的 Kafka 集群中会有用武之地。默认值是 -1，表示可以无限使用磁盘空间。
- max.message.bytes：Broker 能够正常接收该 Topic 的最大消息大小

创建topic时设置:

```

bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic transaction --partitions 1 --replication-factor 1 --config retention.ms=15552000000 --config max.message.bytes=5242880
```

修改topic(推荐)：

```

bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name transaction --alter --add-config max.message.bytes=10485760
```

## JVM参数

 JVM 堆大小设置成 6GB ，用默认的 G1 收集器

- KAFKA_HEAP_OPTS：指定堆大小。
- KAFKA_JVM_PERFORMANCE_OPTS：指定 GC 参数。

启动broker前设置：

```
$> export KAFKA_HEAP_OPTS=--Xms6g  --Xmx6g
$> export KAFKA_JVM_PERFORMANCE_OPTS= -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true
$> bin/kafka-server-start.sh config/server.properties
```

## 操作系统参数

文件描述符限制：`ulimit -n 1000000`，通常将它设置成一个超大的值

文件系统类型：最好选择xfs

Swap：设置成一个比较小的值，当开始使用 swap 空间时，能够观测到 Broker 性能开始出现急剧下降，避免直接OOM，从而给你进一步调优和诊断问题的时间。

提交时间（Flush 落盘时间）：适当地增加提交间隔来降低物理磁盘的写操作