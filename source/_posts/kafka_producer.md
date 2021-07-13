---
title: kafka生产者
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-23 18:45:20
password:
summary:
tags:
- kafka
categories:
- kafka
---

## 消息分区机制

### 为什么分区

**提供负载均衡的能力，实现系统的高伸缩性。**

不同的分区能够被放置到不同节点的机器上，而数据的读写操作也都是针对分区这个粒度而进行的，这样每个节点的机器都能独立地执行各自分区的读写请求处理。并且，还可以通过添加新的节点机器来增加整体系统的吞吐量。

### 分区策略

**自定义分区策略**

编写一个具体的类实现`org.apache.kafka.clients.producer.Partitioner`接口。其中包含`partition()`和`close()`，一般只需要实现 `partition `方法。

**轮询策略**

` Round-robin` 策略，即顺序分配。

轮询策略有非常优秀的负载均衡表现，它总是能保证消息最大限度地被平均分配到所有分区上，最合理也最常用的分区策略，。

**随机策略**

`Randomness `策略。将消息放置到任意一个分区上。

**按消息键保序策略**

Kafka 允许为每条消息定义消息键，简称为 Key。可以保证同一个 Key 的所有消息都进入到相同的分区里面，每个分区下的消息处理都是有顺序的。

```java
List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
return Math.abs(key.hashCode()) % partitions.size();

//可以根据 Broker 所在的 IP 地址实现定制化的分区策略
List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
return partitions.stream().filter(
    p -> isSouth(p.leader().host())
).map(PartitionInfo::partition).findAny().get();
```

## 压缩

Kafka 的消息层次分为两层：消息集合（message set）以及消息（message）。一个消息集合中包含若干条日志项（record item），日志项是真正封装消息的地方。Kafka 通常不会直接操作具体的一条条消息，它总是在消息集合这个层面上进行写入操作。

### V1 版本和 V2 版本区别

1、把消息的公共部分抽取出来放到外层消息集合里面，这样就不用每条消息都保存这些信息了。

2、消息的 CRC 校验工作就被移到了消息集合这一层。

3、 V1 版本中保存压缩消息的方法是把多条消息进行压缩然后保存到外层消息的消息体字段中；而 V2 版本是对整个消息集合进行压缩

### 何时压缩

1、Producer 启动后生产的每个消息集合都是经过压缩的，故而能很好地节省网络传输带宽以及 Kafka Broker 端的磁盘占用。

```java
 Properties props = new Properties();
 props.put("bootstrap.servers", "localhost:9092");
 props.put("acks", "all");
 props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
 props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
 // 开启GZIP压缩
 props.put("compression.type", "gzip");
 
 Producer<String, String> producer = new KafkaProducer<>(props);
```

2、Broker 重新压缩消息

- Broker 端指定了和 Producer 端不同的压缩算法，需要重新解压缩 / 压缩操作。

- Broker 端发生了消息格式转换（V1/V2），涉及消息的解压缩和重新压缩，丧失了的 Zero Copy 特性

### 解压缩

通常来说解压缩发生在消费者程序中。

 Producer 发送压缩消息到 Broker 后，Broker 照单全收并原样保存起来。当 Consumer 程序请求这部分消息时，Broker 依然原样发送出去，当消息到达 Consumer 端后，由 Consumer 自行解压缩还原成之前的消息。

### 压缩算法

吞吐量：LZ4 > Snappy > zstd 和 GZIP；

压缩比：zstd > LZ4 > GZIP > Snappy

## Kafka 生产者程序

第 1 步：构造生产者对象所需的参数对象。

第 2 步：利用参数对象，创建 KafkaProducer 对象实例。

第 3 步：使用 KafkaProducer 的 send 方法发送消息。

第 4 步：调用 KafkaProducer 的 close 方法关闭生产者并释放各种系统资源。

```java
Properties props = new Properties ();
props.put(“参数1”, “参数1的值”)；
props.put(“参数2”, “参数2的值”)；
……
try (Producer<String, String> producer = new KafkaProducer<>(props)) {
    producer.send(new ProducerRecord<String, String>(……), callback);
  ……
}
```

## 生产者的TCP

在创建 KafkaProducer 实例时，生产者应用会在后台创建并启动一个名为 Sender 的线程，该 Sender 线程开始运行时首先会**创建与 Broker 的TCP连接**。它会连接 `bootstrap.servers` 参数指定的所有 Broker。

`KafkaProducer `实例创建的线程和前面提到的 Sender 线程共享的可变数据结构只有 `RecordAccumulator `类，维护了 `RecordAccumulator` 类的线程安全，也就实现了 KafkaProducer 类的线程安全。

**其他TCP连接创建**

- 当 Producer 更新了集群的元数据信息之后，如果发现与某些 Broker 当前没有连接，那么它就会创建一个 TCP 连接。

- 当要发送消息时，Producer 发现尚不存在与目标 Broker 的连接，也会创建一个。

**更新元数据场景**

1、当 Producer 尝试给一个不存在的主题发送消息时，Broker 返回不存在。 Producer 会发送 METADATA 请求给 Kafka 集群，去尝试获取最新的元数据信息。

2、Producer 通过 metadata.max.age.ms 参数定期地去更新元数据信息。该参数的默认值 5 分钟。

**关闭TCP连接**

- 用户主动关闭

- Kafka 自动关闭

connections.max.idle.ms范围内没有任何请求“流过”某个 TCP 连接，那么 Kafka 会主动帮你把该 TCP 连接关闭。

可以在 Producer 端设置 connections.max.idle.ms=-1 ，TCP 连接将成为永久长连接。

## 幂等性 Producer

设置`props.put(“enable.idempotence”, ture)`，或 `props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG，true)`

**底层原理：**用空间去换时间的优化思路，即在 Broker 端多保存一些字段。当 Producer 发送了具有相同字段值的消息后，Broker 能够自动知晓这些消息已经重复了，于是可以在后台默默地把它们“丢弃”掉。

- ProducerID：在每个新的Producer初始化时，会被分配一个唯一的ProducerID，这个ProducerID对客户端使用者是不可见的。
- SequenceNumber：对于每个ProducerID，Producer发送数据的每个Topic和Partition都对应一个从0开始单调递增的SequenceNumber值。

**作用范围：**它只能保证单分区上的幂等性，即一个幂等性 Producer 能够保证某个主题的一个分区上不出现重复消息，它无法实现多个分区的幂等性。其次，它只能实现单会话上的幂等性，不能实现跨会话的幂等性。

多分区以及多会话上的消息无重复，需要**事务**或者依赖**事务型 Producer**

