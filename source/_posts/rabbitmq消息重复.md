---
title: rabbitmq消息重复
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-26 22:18:39
password:
summary:
tags:
- rabbitmq
categories:
- rabbitmq
---

## 场景

- 可靠性投递机制：mq收到生产者消息，mq在返回confirm的时候网络出现闪断，导致broker未收到应答，导致发送两次。
- MQ Broker服务与消费端传输消息的过程中出现网络抖动。
- 消费端故障、异常。

## 解决方案

### 可靠性投递解决

对每条消息，MQ系统内部必须生成一个inner-msg-id，作为去重和幂等的依据，这个内部消息ID的特性是：
（1）全局唯一
（2）MQ生成，具备业务无关性，对消息发送方和消息接收方屏蔽
有了这个inner-msg-id，就能保证上半场重发，也只有1条消息落到MQ-server的DB中，实现上半场幂等。

### 消费抖动解决
业务消息体中，必须有一个biz-id，作为去重和幂等的依据，这个业务ID的特性是：
（1）对于同一个业务场景，全局唯一
（2）由业务消息发送方生成，业务相关，对MQ透明
（3）由业务消息消费方负责判重，以保证幂等