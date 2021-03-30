---
title: Kafka控制器
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-28 19:48:00
password:
summary:
tags:
- kafka
categories:
- kafka
---

控制器组件（Controller），是 Apache Kafka 的核心组件。它的主要作用是在 Apache ZooKeeper 的帮助下管理和协调整个 Kafka 集群。每个正常运转的 Kafka 集群，在任意时刻都有且只有一个控制器。

[](zookeeper.jpg)

## **控制器选举**

Broker 在启动时，会尝试去 ZooKeeper 中创建 /controller 节点。Kafka 当前选举控制器的规则是：第一个成功创建 /controller 节点的 Broker 会被指定为控制器。

## **控制器作用**

- 主题管理（创建、删除、增加分区）
- 分区重分配
- Preferred 领导者选举
- 集群成员管理（新增 Broker、Broker 主动关闭、Broker 宕机）：利用 Watch 机制检测变更。
- 数据服务

[](data.jpg)

## **控制器故障转移**

故障转移指的是，当运行中的控制器突然宕机或意外终止时，Kafka 能够快速地感知到，并立即启用备用控制器来代替之前失败的控制器。这个过程就被称为 Failover，该过程是自动完成的，无需你手动干预。

[](Failover.jpg)

