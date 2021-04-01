---
title: kafka拦截器
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-23 19:18:48
password:
summary:
tags:
- kafka
categories:
- kafka
---

Kafka 拦截器分为生产者拦截器和消费者拦截器。

生产者拦截器允许你在发送消息前以及消息提交成功后植入你的拦截器逻辑；

而消费者拦截器支持在消费消息前以及提交位移后编写特定逻辑。

**使用**

当前 Kafka 拦截器的设置方法是通过参数配置完成的。生产者和消费者两端有一个相同的参数，名字叫` interceptor.classes`，它指定的是一组类的列表，每个类就是特定逻辑的拦截器实现类。

```java
Properties props = new Properties();
List<String> interceptors = new ArrayList<>();
interceptors.add("com.yourcompany.kafkaproject.interceptors.AddTimestampInterceptor"); // 拦截器1
interceptors.add("com.yourcompany.kafkaproject.interceptors.UpdateCounterInterceptor"); // 拦截器2
props.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, interceptors);
```

`AddTimeStampInterceptor` 和 `UpdateCounterInterceptor `这两个类以及你自己编写的所有 Producer 端拦截器实现类都要继承 `org.apache.kafka.clients.producer.ProducerInterceptor `接口。该接口是 Kafka 提供的，里面有两个核心的方法。

- `onSend`：该方法会在消息发送之前被调用。如果你想在发送之前对消息“美美容”。
- `onAcknowledgement`：该方法会在消息成功提交或发送失败之后被调用。onAcknowledgement 的调用要早于 callback 的调用。



消费者拦截器具体的实现类要实现 `org.apache.kafka.clients.consumer.ConsumerInterceptor` 接口，这里面也有两个核心方法。

- `onConsume`：该方法在消息返回给 Consumer 程序之前调用。也就是说在开始正式处理消息之前，拦截器会先拦一道，搞一些事情，之后再返回给你。
- `onCommit`：Consumer 在提交位移之后调用该方法。通常你可以在该方法中做一些记账类的动作，比如打日志等。

**场景**

Kafka 拦截器可以应用于包括客户端监控、端到端系统性能检测、消息审计等多种功能在内的场景。

如：业务消息从被生产出来到最后被消费的平均总时长统计

```java
// 生产者
public class AvgLatencyProducerInterceptor implements ProducerInterceptor<String, String> {


    private Jedis jedis; // 省略Jedis初始化


    @Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
        jedis.incr("totalSentMessage");
        return record;
    }


    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
    }


    @Override
    public void close() {
    }


    @Override
    public void configure(Map<java.lang.String, ?> configs) {
    }
 //消费者
    

public class AvgLatencyConsumerInterceptor implements ConsumerInterceptor<String, String> {


    private Jedis jedis; //省略Jedis初始化


    @Override
    public ConsumerRecords<String, String> onConsume(ConsumerRecords<String, String> records) {
        long lantency = 0L;
        for (ConsumerRecord<String, String> record : records) {
            lantency += (System.currentTimeMillis() - record.timestamp());
        }
        jedis.incrBy("totalLatency", lantency);
        long totalLatency = Long.parseLong(jedis.get("totalLatency"));
        long totalSentMsgs = Long.parseLong(jedis.get("totalSentMessage"));
        jedis.set("avgLatency", String.valueOf(totalLatency / totalSentMsgs));
        return records;
    }


    @Override
    public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {
    }


    @Override
    public void close() {
    }


    @Override
    public void configure(Map<String, ?> configs) {
    }
}
```

