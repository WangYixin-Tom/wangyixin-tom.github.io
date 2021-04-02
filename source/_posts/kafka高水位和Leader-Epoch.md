---
title: kafka高水位和Leader Epoch
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-28 20:07:34
password:
summary:
tags:
- kafka
categories:
- kafka
---

## 高水位

在分区高水位以下的消息被认为是已提交消息。kafka中，分区的高水位就是其 Leader 副本的高水位。

**作用**

- 定义消息可见性，即用来标识分区下的哪些消息是可以被消费者消费的。
- 帮助 Kafka 完成副本同步。

**LEO（Log End Offset）**表示副本写入下一条消息的位移值。

## 高水位更新机制

![](water.jpg)

![](watertime.jpg)

### **Leader 副本高水位**

**处理生产者请求**的逻辑如下：

1、写入消息到本地磁盘。

2、更新分区高水位值。

i. 获取 Leader 副本所在 Broker 端保存的所有远程副本 LEO 值（LEO-1，LEO-2，……，LEO-n）。

ii. 获取 Leader 副本高水位值：currentHW。

iii. 更新 `currentHW = max{currentHW, min（LEO-1, LEO-2, ……，LEO-n）}`。

**处理 Follower 副本拉取消息**的逻辑如下：

1、读取磁盘（或页缓存）中的消息数据。

2、使用 Follower 副本发送请求中的位移值更新远程副本 LEO 值。

3、更新分区高水位值（具体步骤与处理生产者请求的步骤相同）。

### **Follower 副本高水位**

**从 Leader 拉取消息的处理逻辑**如下：

1、写入消息到本地磁盘。

2、更新 LEO 值。

3、更新高水位值。

i. 获取 Leader 发送的高水位值：currentHW。

ii. 获取步骤 2 中更新过的 LEO 值：currentLEO。

iii. 更新高水位为 `min(currentHW, currentLEO)`。

### 高水位更新说明

新消息写入时，先更新leader副本LEO，

follower副本新消息写入后第一次拉消息，更新了follower副本的LEO，

follower第二次拉消息，leader副本更新remote LEO、HW；follower副本更新高水位

**问题**：Follower 端高水位的更新与 Leader 端有时间错配。如果在这个短暂的滞后时间窗口内，接连发生 Broker 宕机，可能发生数据丢失。

**背景**：副本 A 和副本 B 都处于正常状态，A 是 Leader 副本。某个使用了默认 acks 设置的生产者程序向 A 发送了两条消息，A 全部写入成功，此时 Kafka 会通知生产者说两条消息全部发送成功。

![](bad.jpg)

1、副本 B 所在的 Broker 宕机，当它重启回来后，副本 B 会执行日志截断操作，将 **LEO 值由2调整为之前的高水位值**，也就是 1。

2、副本 B 开始从 A 拉取消息前，副本 A 所在的 Broker 宕机了，副本 B 成为新的 Leader，A 回来后，需要执行相同的日志截断操作，即将高水位调整为与 B 相同的值，也就是 1。

**影响：**位移值为 1 的消息丢失。

## Leader Epoch

它由两部分数据组成。

- Epoch。一个单调增加的版本号。每当副本领导权发生变更时，都会增加该版本号。小版本号的 Leader 被认为是过期 Leader，不能再行使 Leader 权力。
- 起始位移（Start Offset）。Leader 副本在该 Epoch 值上写入的首条消息的位移。

Kafka Broker 会在内存中为每个分区都缓存 Leader Epoch 数据，同时它还会定期地将这些信息持久化到一个 checkpoint 文件中。当 Leader 副本写入消息到磁盘时，Broker 会尝试更新这部分缓存。如果该 Leader 是首次写入消息，那么 Broker 会向缓存中增加一个 Leader Epoch 条目。

**解决：**

![](good.jpg)

Follower 副本 B 重启回来后，需要向 A 发送一个特殊的请求去获取 **Leader 的 LEO 值**。在这个例子中，该值为 2。当获知到 Leader LEO=2 后，B 发现该 LEO 值不比它自己的 LEO 值小，而且缓存中也没有保存任何起始位移值 > 2 的 Epoch 条目，因此 B 无需执行任何日志截断操作。

副本 A 宕机了，B 成为 Leader。同样地，当 A 重启回来后，执行与 B 相同的逻辑判断，发现也不用执行日志截断，至此位移值为 1 的那条消息在两个副本中均得到保留。

后面当生产者程序向 B 写入新消息时，副本 B 所在的 Broker 缓存中，会生成新的 Leader Epoch 条目：[Epoch=1, Offset=2]。之后，副本 B 会使用这个条目帮助判断后续是否执行日志截断操作。