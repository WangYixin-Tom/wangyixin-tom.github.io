---
title: CAP和BASE理论
top: false
cover: false
toc: true
mathjax: true
date: 2021-06-23 21:41:02
password:
summary:
tags:
- 分布式
categories:
- 分布式
---

## CAP定理

一个分布式系统不可能同时满足一致性（C:Consistency)，可用性（A: Availability）和分区容错性（P：Partition tolerance）这三个基本需求，最多只能同时满足其中的2个。

**一致性**，指数据在多个副本之间能够保持一致的特性（严格的一致性）。

**可用性**，指系统提供的服务必须一直处于可用的状态，每次请求总能在**有限时间内返回结果**。

**分区容错性**，分布式系统在遇到任何网络分区故障的时候，仍然能够对外提供满足一致性和可用性的服务，除非整个网络环境都发生了故障。

### 为什么只能3选2

整个系统由两个节点配合组成，之间通过网络通信，当节点 A 进行更新数据库操作的时候，需要同时更新节点 B 的数据库（这是一个原子的操作）。

当节点 A,B 出现了网络分区，

- 当节点A更新的时候，节点B也要更新
- 必须保证两个节点都是可用的。

显然无法满足，A节点无法连上B节点。如果一定要满足一致性，就必须放弃可用性，等待网络恢复。

### 应用

放弃P：将所有数据放到一个分布式节点上，放弃P，也放弃了系统的扩展性

放弃A：收到影响的服务需要等待一定时间，在此期间服务不可用

放弃C：放弃数据的强一致性，保留最终一致性。虽然无法保证实时的一致性，但是数据最终会达到一个一致的状态。

## BASE理论

**BASE：**全称：Basically Available(基本可用)，Soft state（软状态）和 Eventually consistent（最终一致性）三个短语的缩写

**核心思想：**即使无法做到强一致性（Strong consistency），但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）。

### 基本可用

出现了不可预知的故障，损失部分可用性，如：

- 响应时间上的损失：正常情况下的搜索引擎 0.5 秒即返回给用户结果，出现故障后，查询结果响应时间增加到了1-2秒。
- 功能上的损失：在一个电商网站上，正常情况下，用户可以顺利完成每一笔订单，但是到了大促期间，为了保护购物系统的稳定性，

### 软状态

软状态指的是：允许系统中的数据存在中间状态，并认为该状态不影响系统的整体可用性，即允许系统在多个不同节点的数据副本之间数据同步存在一定延时。

### 最终一致性

系统能够保证在没有其他新的更新操作的情况下，数据最终一定能够达到一致的状态，因此所有客户端对系统的数据访问最终都能够获取到最新的值。

**最终一致性分为 5 种：**

**1. 因果一致性（Causal consistency）**

如果进程 A 在更新完某个数据后通知了进程 B，那么进程B 之后对该数据的访问和修改都是基于 A 更新后的值。于此同时，和进程A 无因果关系的进程C 的数据访问则没有这样的限制。

**2. 读己之所写（Read your writes）**

这种就很简单了，进程A 更新一个数据后，它自身总是能访问到自身更新过的最新值，而不会看到旧值。其实也算一种因果一致性。

**3. 会话一致性（Session consistency）**

会话一致性将对系统数据的访问过程框定在了一个会话当中：系统能保证在同一个有效的会话中实现 “读己之所写” 的一致性，也就是说，执行更新操作之后，客户端能够在同一个会话中始终读取到该数据项的最新值。

**4. 单调读一致性（Monotonic read consistency）**

单调读一致性是指如果一个节点从系统中读取出一个数据项的某个值后，那么系统对于该节点后续的任何数据访问都不应该返回更旧的值。

**5. 单调写一致性（Monotonic write consistency）**

指一个系统要能够保证来自同一个节点的写操作被顺序的执行。

### 总结

BASE 理论面向的是大型高可用可扩展的分布式系统，和传统事务的 ACID 是**相反的**，它完全不同于 ACID 的强一致性模型，而是**通过牺牲强一致性**来获得可用性，并允许数据在一段时间是不一致的。