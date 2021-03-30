---
title: kafka主题管理
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-28 21:49:14
password:
summary:
tags:
- kafka
categories:
- kafka
---

## **主题增删改查**

### 创建

```shell
bin/kafka-topics.sh --bootstrap-server broker_host:port --create --topic my_topic_name  --partitions 1 --replication-factor 1
```

从 Kafka 2.2 版本开始，社区推荐用 --bootstrap-server 参数替换 --zookeeper 参数

### 查询

```shell
# 查询所有主题的列表
bin/kafka-topics.sh --bootstrap-server broker_host:port --list
# 查询单个主题的详细数据
bin/kafka-topics.sh --bootstrap-server broker_host:port --describe --topic <topic_name>
```

### 修改

```shell
# 增加分区
bin/kafka-topics.sh --bootstrap-server broker_host:port --alter --topic <topic_name> --partitions <新分区数>

# 修改主题级别参数,常规的主题级别参数，使用 --zookeeper;动态参数使用 --bootstrap-server 
bin/kafka-configs.sh --zookeeper zookeeper_host:port --entity-type topics --entity-name <topic_name> --alter --add-config max.message.bytes=10485760

# 变更副本数
# reassign.json
{"version":1, "partitions":[
 {"topic":"__consumer_offsets","partition":0,"replicas":[0,1,2]}, 
  {"topic":"__consumer_offsets","partition":1,"replicas":[0,2,1]},
  {"topic":"__consumer_offsets","partition":2,"replicas":[1,0,2]},
  {"topic":"__consumer_offsets","partition":3,"replicas":[1,2,0]},
  ...
  {"topic":"__consumer_offsets","partition":49,"replicas":[0,1,2]}
]}` 
bin/kafka-reassign-partitions.sh --zookeeper zookeeper_host:port --reassignment-json-file reassign.json --execute

 # 修改test主题限速
bin/kafka-configs.sh --zookeeper zookeeper_host:port --alter --add-config 'leader.replication.throttled.rate=104857600,follower.replication.throttled.rate=104857600' --entity-type brokers --entity-name 0
bin/kafka-configs.sh --zookeeper zookeeper_host:port --alter --add-config 'leader.replication.throttled.replicas=*,follower.replication.throttled.replicas=*' --entity-type topics --entity-name test

# 主题分区迁移
kafka-reassign-partitions
```

### 删除

```shell
bin/kafka-topics.sh --bootstrap-server broker_host:port --delete  --topic <topic_name>
```

## 特殊主体管理

### 查看消费者组提交的位移数

```shell
bin/kafka-console-consumer.sh --bootstrap-server kafka_host:port --topic __consumer_offsets --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" --from-beginning
```

### 查看消费者组的状态信息

```shell
bin/kafka-console-consumer.sh --bootstrap-server kafka_host:port --topic __consumer_offsets --formatter "kafka.coordinator.group.GroupMetadataManager\$GroupMetadataMessageFormatter" --from-beginning
```

## 常见主题错误处理

### 主题删除失败

- 副本所在的 Broker 宕机了；--重启即可
- 待删除主题的部分分区依然在执行迁移过程

**解决**

第 1 步，手动删除 ZooKeeper 节点 /admin/delete_topics 下以待删除主题为名的 znode。

第 2 步，手动删除该主题在磁盘上的分区目录。

第 3 步，在 ZooKeeper 中执行 rmr  /controller，触发 Controller 重选举，刷新 Controller 缓存。--非必须，可能造成大面积的分区 Leader 重选举

### _consumer_offsets 占用太多的磁盘

 jstack 命令查看一下 kafka-log-cleaner-thread 前缀的线程状态。通常情况下，这都是因为该线程挂掉了，无法及时清理此内部主题。

