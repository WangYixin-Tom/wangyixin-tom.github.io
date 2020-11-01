---
title: rabbitmq集群
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-26 22:17:41
password:
summary:
tags:
- rabbitmq
categories:
- rabbitmq
---

## 集群概述

------------

### 集群节点类型

- 磁盘节点：运行时状态信息（集群、队列、绑定虚拟主机、用户、策略等）存储在内存和磁盘中。集群至少有一个磁盘节点。关闭集群后，重启时需要按照一定顺序启动。
- 内存节点：运行时状态信息（集群、队列、绑定虚拟主机、用户、策略等）存储在内存中。重启加入集群是，需要从其他节点同步。

**统计节点**
rabbitmq管理插件包含，必须搭配磁盘节点，且一个集群只能有一个统计节点。统计节点负责收集每个节点的全部统计数据和状态数据。主节点（统计节点）故障，备用磁盘节点将被指定为统计节点。

## 集群设置

**加入集群**

```
1、在第一节点上运行rabbitmq
2、在第二节点上停止rabbitmq，并清除状态
rabbitmqctl stop_app
rabbitmqctl reset
3、加入主节点，构成集群
rabbitmqctl join_cluster rabbitmq@node1
4、再次启动第二节点rabbitmq
rabbitmqctl start_app



```