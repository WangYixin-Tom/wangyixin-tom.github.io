---
title: kafka调优
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-30 21:30:58
password:
summary:
tags:
- kafka
categories:
- kafka
---

## 调优目标

吞吐量，也就是 TPS，是指 Broker 端进程或 Client 端应用程序每秒能处理的字节数或消息数。

延时，表示从 Producer 端发送消息到 Broker 端持久化完成之间的时间间隔。

## 优化

### 操作系统调优

1、最好在挂载（Mount）文件系统时禁掉 atime 更新。atime 的全称是 access time，记录的是文件最后被访问的时间。可以避免 inode 访问时间的写入操作，减少文件系统的写操作数。

2、建议将 swappiness 设置成一个很小的值，比如 1～10 之间，以防止 Linux 的 OOM Killer 开启随意杀掉进程

3、ulimit -n 不宜太小

4、给 Kafka 预留的页缓存越大越好，预留出一个日志段大小，至少能保证 Kafka 可以将整个日志段全部放入页缓存，这样，消费者程序在消费时能直接命中页缓存，从而避免昂贵的物理磁盘 I/O 操作。

### JVM 层调优

1. 设置堆大小。6～8GB
2. GC 收集器使用 G1 收集器。

### Broker 端调优

保持客户端版本和 Broker 端版本一致。

### 应用层调优

1、不要频繁地创建 Producer 和 Consumer 对象实例。构造这些对象的开销很大，尽量复用它们。

2、用完及时关闭。这些对象底层会创建很多物理资源，如 Socket 连接、ByteBuffer 缓冲区等。不及时关闭的话，势必造成资源泄露。3、合理利用多线程来改善性能。生产者、消费者多线程。

## 性能指标调优

### 调优吞吐量

1、broker 端参数 `num.replica.fetchers `表示的是 Follower 副本用多少个线程来拉取消息，默认使用 1 个线程。如果你的 Broker 端 CPU 资源很充足，不妨适当调大该参数值，加快 Follower 副本的同步速度。

2、在 Producer 端，如果要改善吞吐量，通常的标配是增加消息批次的大小以及批次缓存时间，即 `batch.size` 和 `linger.ms`。

3、压缩算法也配置上，以减少网络 I/O 传输量，从而间接提升吞吐量。当前，和 Kafka 适配最好的两个压缩算法是 LZ4 和 zstd

4、Consumer端使用多线程方案

### 调优延时

1、在 Broker 端，我们依然要增加 `num.replica.fetchers` 值以加快 Follower 副本的拉取速度，减少整个消息处理的延时

2、设置 `linger.ms=0`，同时不要启用压缩。因为压缩操作本身要消耗 CPU 时间。

3、Consumer 端，我们保持` fetch.min.bytes=1` 即可，也就是说，只要 Broker 端有能返回的数据，立即令其返回给 Consumer，缩短 Consumer 消费延时。