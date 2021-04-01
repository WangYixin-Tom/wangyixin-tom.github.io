---
title: kafka精确一次
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-23 20:59:45
password:
summary:
tags:
- kafka
categories:
- kafka
---

kafka通过**幂等性（Idempotence）**和**事务（Transaction）**实现消息精确一次（exactly once）的可靠性保障。

## 幂等性 Producer

设置`props.put(“enable.idempotence”, ture)`，或` props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG， true)`

**底层原理：**用空间去换时间的优化思路，即在 Broker 端多保存一些字段。当 Producer 发送了具有相同字段值的消息后，Broker 能够自动知晓这些消息已经重复了，于是可以在后台默默地把它们“丢弃”掉。

**作用范围：**它只能保证单分区上的幂等性，即一个幂等性 Producer 能够保证某个主题的一个分区上不出现重复消息，它无法实现多个分区的幂等性。其次，它只能实现单会话上的幂等性，不能实现跨会话的幂等性。

多分区以及多会话上的消息无重复，需要依赖**事务型 Producer**.

## **事务型 Producer**

事务型 Producer 能够保证将消息原子性地写入到多个分区中。这批消息要么全部写入成功，要么全部失败。

设置事务型 Producer 的方法：

- 开启`enable.idempotence = true`。

- 设置 Producer 端参数` transactional.id`。最好为其设置一个有意义的名字。

```java
producer.initTransactions();
try {
    producer.beginTransaction();
    producer.send(record1);
    producer.send(record2);
    producer.commitTransaction();
} catch (KafkaException e) {
    producer.abortTransaction();
}
```

`isolation.level`支持`read_uncommitted`和`read_committed`