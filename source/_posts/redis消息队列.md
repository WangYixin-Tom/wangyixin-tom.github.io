---
title: redis消息队列
top: false
cover: false
toc: true
mathjax: true
date: 2020-11-02 21:35:36
password:
summary:
tags:
- redis
categories:
- redis
---

## 需求分析

- 消息保序：消费者需要按照生产者发送消息的顺序来处理消息

- 处理重复的消息：消费者避免多次处理重复的消息

- 保证消息可靠性：消费者重启后，可以重新读取消息再次进行处理

## 基于List

-  LPUSH 

把要发送的消息依次写入 List

- BRPOP 

阻塞式读取，客户端在没有读到队列数据时，自动阻塞，直到有新的数据写入队列。

- 消费者程序本身能对重复消息进行判断

消息队列要能给每一个消息提供全局唯一的 ID 号消费者程序要把已经处理过的消息的 ID 号记录下来。ID号需要生产者程序在发送消息前自行生成，并在LPUSH的时候插入List。

- BRPOPLPUSH

让消费者程序从一个 List 中读取消息，同时，把这个消息再插入到另一个 List（可以叫作备份 List）留存

缺点：不支持消费组

## 基于 Streams（Redis 5.0）

- XADD：插入消息，保证有序，可以自动生成全局唯一 ID；
- XREAD：用于读取消息，可以按 ID 读取数据；
- XREADGROUP：按消费组形式读取消息
- XPENDING 和 XACK：XPENDING 命令可以用来查询每个消费组内所有消费者已读取但尚未确认的消息，而 XACK 命令用于向消息队列确认消息处理已完成。

## 缺点

在用Redis当作队列或存储数据时，是有可能丢失数据的：AOF同步写盘会降低性能。主从集群切换也可能丢数据。

