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

## Leader Epoch

它由两部分数据组成。

- Epoch。一个单调增加的版本号。每当副本领导权发生变更时，都会增加该版本号。小版本号的 Leader 被认为是过期 Leader，不能再行使 Leader 权力。
- 起始位移（Start Offset）。Leader 副本在该 Epoch 值上写入的首条消息的位移。