---
title: KafkaAdminClient
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-29 20:54:48
password:
summary:
tags:
- kafka
categories:
- kafka
---

## 功能

主题管理：包括主题的创建、删除和查询。

权限管理：包括具体权限的配置与删除。

配置参数管理：包括 Kafka 各种资源的参数设置、详情查询。所谓的 Kafka 资源，主要有 Broker、主题、用户、Client-id 等。

副本日志管理：包括副本底层日志路径的变更和详情查询。

分区管理：即创建额外的主题分区。

消息删除：即删除指定位移之前的分区消息。

Delegation Token 管理：包括 Delegation Token 的创建、更新、过期和详情查询。

消费者组管理：包括消费者组的查询、位移查询和删除。

Preferred 领导者选举：推选指定主题分区的 Preferred Broker 为领导者。

## 工作原理

AdminClient 是一个双线程的设计：前端主线程和后端 I/O 线程。

前端线程负责将用户要执行的操作转换成对应的请求，然后再将请求发送到后端 I/O 线程的队列中；

而后端 I/O 线程(kafka-admin-client-thread)从队列中读取相应的请求，然后发送到对应的 Broker 节点上，之后把执行结果保存起来，以便等待前端线程的获取。

[](yuanli.jpg)

## 应用

### 创建实例

```java
Properties props = new Properties();
props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka-host:port");
props.put("request.timeout.ms", 600000);

try (AdminClient client = AdminClient.create(props)) {
    // 执行你要做的操作……
}
```

### 创建主题

```java
String newTopicName = "test-topic";
try (AdminClient client = AdminClient.create(props)) {
    NewTopic newTopic = new NewTopic(newTopicName, 10, (short) 3); //主题名称、分区数和副本数
    CreateTopicsResult result = client.createTopics(Arrays.asList(newTopic));
    result.all().get(10, TimeUnit.SECONDS);
}
```

### 查询消费者组位移

```java
String groupID = "test-group";
try (AdminClient client = AdminClient.create(props)) {
    ListConsumerGroupOffsetsResult result = client.listConsumerGroupOffsets(groupID);
    Map<TopicPartition, OffsetAndMetadata> offsets = 
        result.partitionsToOffsetAndMetadata().get(10, TimeUnit.SECONDS);
    System.out.println(offsets);
}
```

### 获取 Broker 磁盘占用

```java

try (AdminClient client = AdminClient.create(props)) {
    DescribeLogDirsResult ret = client.describeLogDirs(Collections.singletonList(targetBrokerId)); // 指定Broker id
    long size = 0L;
    for (Map<String, DescribeLogDirsResponse.LogDirInfo> logDirInfoMap : ret.all().get().values()) {
        size += logDirInfoMap.values().stream().map(logDirInfo -> logDirInfo.replicaInfos).flatMap(
            topicPartitionReplicaInfoMap ->
            topicPartitionReplicaInfoMap.values().stream().map(replicaInfo -> replicaInfo.size))
            .mapToLong(Long::longValue).sum();
    }
    System.out.println(size);
}
```

