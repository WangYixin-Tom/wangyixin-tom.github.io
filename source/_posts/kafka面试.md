---
title: kafka面试
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-08 19:55:21
password:
summary:
tags:
- interview
categories:
- interview
---

## Kafka 是什么？主要应用场景有哪些？

Kafka 是一个分布式流式处理平台，可以作为企业级的消息引擎。

Kafka 主要有两大应用场景：

1. **消息队列** ：建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。
2. **数据处理：** 构建实时的流数据处理程序来转换或处理数据流。

## Kafka的优势在哪里？

1. **极致的性能** ：最高可以每秒处理千万级别的消息。
2. **生态系统兼容性无可匹敌** ：尤其在大数据和流计算领域。

## 队列模型？Kafka 的消息模型？

### 队列模型

使用队列作为消息通信载体，一条消息只能被一个消费者使用，未被消费的消息在队列中保留直到被消费或超时。 

**问题**

将生产者产生的消息分发给多个消费者难以处理。

### 发布-订阅模型（Kafka 消息模型）

 使用主题作为消息通信载体，类似于广播模式；生产者发布一条消息，该消息通过主题传递给所有的订阅者。

如果只有一个订阅者，那就退化到队列模型。

## Producer、Consumer、Broker、Topic、Partition、Record？

1. **Producer（生产者）** ： 产生消息的一方。
2. **Consumer（消费者）**：消费消息的一方。
3. **Broker（代理）** : 可以看作是一个独立的 Kafka 实例。多个 Kafka Broker 组成一个 Kafka Cluster。

- **Topic（主题）** : kafka 通过不同的主题区分不同的业务类型的消息记录。Producer 将消息发送到特定的主题，Consumer 通过订阅特定的主题来消费消息。
- **Partition（分区）**： Partition属于Topic 的一部分。一个 Topic 可以有多个 Partition ，并且同一 Topic 下的 Partition 可以分布在不同的 Broker 上。
- **记录（Record）：**实际写入到kafka集群并且可以被消费者读取的数据。每条记录包含一个键、值和时间戳。

## 什么是消费者组？

可扩展且具有容错性的消费者机制。

- Kafka允许你将同一份消息广播到多个消费者组里。
- 一个消费者组中可以包含多个消费者，他们共同消费该主题的数据。
- 同一个消费者组下的消费者有相同的组ID，他们被分配不同的订阅分区。
- 当某个消费者挂掉的时候，其他消费者会自动地承担起它负责消费的分区。 

## LEO、LSO、AR、ISR、HW？

- LEO（Log End Offset）：日志末端位移值或末端偏移量，表示日志下一条待插入消息的位移值。
- LSO（Log Stable Offset）：该值控制了事务型消费者能够看到的消息范围。
- AR（Assigned Replicas）：主题被创建后，创建的副本集合，副本个数由副本因子决定。
- ISR（In-Sync Replicas）：AR中与Leader保持同步的副本集合。Leader副本天然在ISR中。
- HW（High watermark）：高水位值，这是控制消费者可读取消息范围。一个普通消费者只能看到Leader副本上介于Log Start Offset和HW（不含）之间的所有消息。

## 位移的作用

用于标识消息在分区中的位置。

一旦消息被写入到分区日志，它的位移值将不能被修改。

## Kafka数据索引如何实现

- kafka解决查询效率的手段之一是将数据文件分段存储。每一个段单独放在一个.log的文件中，数据文件命名是20个字符的长度，以每一个分段文件开始的最下offset来命名，其他位置用0填充。
- 每一个log文件的大小默认是1GB，每一个log文件就会对应一个index文件，是和log文件的命名相同的。
- index文件中并没有为每一条message建立索引。而是采用了稀疏存储的方式，每隔一定字节的数据建立一条索引，避免了索引文件占用过多的空间和资源，从而可以将索引文件保留到内存中。缺点是没有建立索引的数据在查询的过程中需要小范围内的顺序扫描操作。

## 磁盘容量规划考虑因素？

  - 新增消息数
  - 消息留存时间
  - 平均消息大小
  - 备份数
  - 是否启用压缩

## __consumer_offsets 作用？

- 内部主题，存储消费者的位移数据
- 保存消费者组相关的消息
- 用于删除消费者组过期位移、删除消费者组的消息。tombstone 消息，即墓碑消息

## 多副本机制？好处？

- Kafka副本当前分为领导者副本和追随者副本。每个分区在创建时都要选举一个副本，称为领导者副本，其余的副本自动称为追随者副本。
- 只有Leader副本才能对外提供读写服务。
- Follower副本只是采用拉的方式，同步Leader副本中的数据
- 在Leader副本所在的Broker宕机后，Kafka 依托于 ZooKeeper 提供的监控，实时感知到，并立即开启新一轮的领导者选举，从追随者副本中选一个作为新的领导者。

**多分区多副本好处？**

1. Kafka 通过给特定 Topic 指定多个 Partition， 而各个 Partition 可以分布在不同的 Broker 上， 这样便能提供比较好的并发能力（负载均衡）。
2. 多副本提高了消息存储的安全性，提高了容灾能力，不过也相应的增加了所需要的存储空间。
3. 读写走leader：方便实现Read-your-writes
4. 读写走leader：方便实现单调读，在多次消费消息时，它不会看到某条消息一会儿存在一会儿不存在

## Zookeeper 的作用？

- 存放元数据：主题分区的相关数据都保存在 ZooKeeper 中，且以它保存的数据为权威。
- 成员管理：Broker 节点的注册、注销以及属性变更。
- Controller 选举：选举集群 Controller节点
- 其他管理类任务：包括但不限于主题删除、参数配置等。

##  如何保证消息的消费顺序？

分区是真正保存消息的地方。Topic 可以指定多个 Partition。

Kafka 只能为我们保证分区中的消息有序，而不能保证 主题中的消息有序。

1. 1 个 Topic 只对应一个 Partition。
2. （推荐）发送消息的时候指定 key/Partition。

## 如何保证消息不丢失

### 生产者丢失消息

- 生产者调用`send`方法发送消息之后，消息可能因为网络问题并没有发送过去。 为了确定消息是发送成功，我们要判断消息发送的结果。可以采用回调函数的形式，如果消息发送失败的话，我们检查失败的原因之后重新发送即可。

- 为 生产者的`retries `（重试次数）设置一个比较合理的值，一般是 3 。
- **设置 acks = all**，则表明所有副本 Broker 都要接收到消息，该消息才算是已提交

### 消费者丢失消息

- 关闭自动提交 offset，每次在真正消费完消息之后之后再自己手动提交 offset 。

### Kafka 弄丢消息

- 设置 `replication.factor >= 3`。防止消息丢失的主要机制就是冗余，最好将消息多保存几份。
- 设置 `min.insync.replicas > 1`。消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。
- 设置 `unclean.leader.election.enable = false`。如果一个 Broker 落后原先的 Leader 太多，那么它一旦成为新的 Leader，必然会造成消息的丢失。故一般设置成 false。

## 如何保证消息不重复消费

去重：将消息的唯一标识保存到外部介质中，每次消费处理时判断是否处理过。

## 怎么增加消费的能力？

1、broker 端参数 `num.replica.fetchers `表示的是 Follower 副本用多少个线程来拉取消息，默认使用 1 个线程。如果你的 Broker 端 CPU 资源很充足，不妨适当调大该参数值，加快 Follower 副本的同步速度。

2、在 Producer 端，如果要改善吞吐量，通常的标配是增加消息批次的大小以及批次缓存时间，即 `batch.size` 和 `linger.ms`。

3、压缩算法也配置上，以减少网络 I/O 传输量，从而间接提升吞吐量。当前，和 Kafka 适配最好的两个压缩算法是 LZ4 和 zstd

4、Consumer端使用多线程方案

## 为什么性能好

### 顺序写

> 操作系统读写磁盘时，需要先寻址，再进行数据读写。如果是机械硬盘，寻址就需要较长的时间。

 Kafka 用的是顺序写，追加数据是追加到末尾，磁盘顺序写的性能极高。

### 零拷贝

Kafka 利用了 Linux 的 sendFile 技术实现零拷贝，直接从磁盘文件复制到网卡设备中,减少了内核和用户模式之间的上下文切换。

### 网络线程模型

加强版的 Reactor 网络线程模型

### 消息批量量处理

合并小的请求，然后以流的方式进行交互，直顶网络上限。

## 为什么不支持读写分离？

CAP理论下，我们只能保证可用性和一致性取其一。

如果支持读写分离，一致性就会有一定折扣，意味着可能的数据不一致，或数据滞后。

## Controller发生网络分区时，Kafka会怎么样？

判断：Broker端的ActiveControllerCount。

由于Controller会给Broker发送3类请求，LeaderAndIsrRequest，StopReplicaRequest，UpdateMetadataRequest，因此，一旦出现网络分区，这些请求将不能顺利到达Broker端。

将影响主题的创建、修改、删除操作的信息同步。

## Java Consumer 为什么采用单线程来获取消息？

Java Consumer是双线程的设计。用户主线程，负责获取消息；心跳线程，负责向Kafka汇报消费者存活情况。将心跳单独放入专属的线程，能够有效地规避因消息处理速度慢而被视为下线的“假死”情况。

- 单线程获取消息的设计能够避免阻塞式的消息获取方式。单线程轮询方式容易实现异步非阻塞式，这样便于将消费者扩展成支持实时流处理的操作算子。因为很多实时流处理操作算子都不能是阻塞式的。
- 可以简化代码的开发。多线程交互的代码是非常容易出错的。

## Follower副本消息同步的流程？

- Follower发送FETCH请求给Leader。
- Leader会读取底层日志文件中的消息数据，使用FETCH请求中的fetchOffset，更新Follower远程副本的LEO值。leader副本尝试更新分区高水位值。
- Follower接收到FETCH响应之后，会把消息写入到底层日志，接着更新LEO和HW值。

Leader和Follower的HW值更新时机是不同的，Follower的HW更新永远落后于Leader的HW。这种时间上的错配是造成各种不一致的原因。

### 怎么保证同步成功？

leader Epoch 由两部分数据组成。

- Epoch。一个单调增加的版本号。每当副本领导权发生变更时，都会增加该版本号。小版本号的 Leader 被认为是过期 Leader，不能再行使 Leader 权力。
- 起始位移（Start Offset）。Leader 副本在该 Epoch 值上写入的首条消息的位移。

Kafka Broker 会在内存中为每个分区都缓存 Leader Epoch 数据，同时它还会定期地将这些信息持久化到一个 checkpoint 文件中。当 Leader 副本写入消息到磁盘时，Broker 会尝试更新这部分缓存。如果该 Leader 是首次写入消息，那么 Broker 会向缓存中增加一个 Leader Epoch 条目。

## Leader总是-1，怎么破？

通常情况下就是Controller不工作了，导致无法分配leader。

方法

1、删除ZooKeeper中的/controller节点，触发Controller重选举。Controller重选举能够为所有主题分区重刷分区状态，可以有效解决因不一致导致的 Leader 不可用问题。

2、重启Controller节点上的Kafka进程，让其他节点重新注册Controller角色。

## 如何设置Kafka能接收的最大消息的大小？

- Broker端参数：`message.max.bytes`，`max.message.bytes`（topic级别），`replica.fetch.max.bytes`
- Consumer端参数：`fetch.message.max.bytes`

### Kafka如何保证高可用
Kafka 的副本机制：
- Kafka 集群由多个 Broker组成。一个Broker可以容纳多个Topic，也就是一台服务器可以传输多个 Topic 数据。Kafka 为了实现可扩展性，将一个 Topic 分散到多个 Partition 中。
- Kafka 中同一个分区下的不同副本，分为 Leader和 Follower。Leader 负责处理所有 读写的请求，Follower 作为数据备份，拉取 Leader 副本的数据进行同步。
- 如果某个 Broker 挂掉，Kafka 会从 ISR 列表中选择一个分区作为新的 Leader 副本。

## Kafka能手动删除消息吗？

一般不需要用户手动删除消息。它本身提供了留存策略，能够自动删除过期消息。同时支持手动删除消息的。

- 对于设置了Key且参数cleanup.policy=compact的主题而言，我们可以构造一条消息发送给Broker，依靠日志清理组件提供的功能删除掉该 Key 的消息。
- 对于普通主题，可以使用`kafka-delete-records`，或编写程序调用`Admin.deleteRecords`方法来删除消息。

## 如何确定合适的Kafka主题的分区数量？

需要根据每个分区的生产者和消费者的期望吞吐量进行估计,以便达到并行读写、负载均衡和高吞吐。

> 假设期望读取数据的速率1GB/Sec，而一个消费者的读取速率为50MB/Sec，此时至少需要20个分区以及20个消费者。
>
> 如果期望生产数据的速率为**1GB/Sec**，而每个生产者的生产速率为**100MB/Sec**，此时就需要有10个分区。
>
> 设置20个分区，既可以保障生产速率，也可以保障的吞吐量

## kafka如何实现延迟队列？

**基于时间轮自定义了一个用于实现延迟功能的定时器（SystemTimer）**。基于时间轮可以将插入和删除操作的时间复杂度都降为`O（1）`。

**底层使用数组实现，**数组中的每个元素可以存放一个TimerTaskList对象。TimerTaskList是一个环形双向链表，在其中的链表项TimerTaskEntry中封装了真正的定时任务TimerTask。

**推进时间？**Kafka中的定时器借助了JDK中的DelayQueue来协助推进时间轮。具体做法是对于每个使用到的TimerTaskList都会加入到DelayQueue中。Kafka中的`TimingWheel`专门用来执行插入和删除TimerTaskEntry的操作，而DelayQueue专门负责时间推进的任务。再试想一下，DelayQueue中的第一个超时任务列表的expiration为200ms，第二个超时任务为840ms，这里获取DelayQueue的队头只需要O（1）的时间复杂度。如果采用每秒定时推进，那么获取到第一个超时的任务列表时执行的200次推进中有199次属于“空推进”，而获取到第二个超时任务时有需要执行639次“空推进”，这样会无故空耗机器的性能资源，**这里采用DelayQueue来辅助以少量空间换时间，从而做到了“精准推进”**。Kafka中的定时器真可谓是“知人善用”，用TimingWheel做最擅长的任务添加和删除操作，而用DelayQueue做最擅长的时间推进工作，相辅相成。

##  判断一个节点还活着的两个条件？

（1）节点必须可以维护和 ZooKeeper 的连接，Zookeeper 通过心跳机制检查每个节点的连接

（2）如果节点是 follower，他必须能及时的同步 leader 的写操作，延时不能太久

## 消息是采用 Pull 模式，还是 Push 模式？

生产者将消息推送到 broker，消费者从 broker 拉取消息。

##  存储在硬盘上的消息格式是什么？

消息由一个固定长度的头部和可变长度的字节数组组成。

- 消息长度: 4 bytes
- 版本号: 1 byte
- CRC 校验码: 4 bytes
- 具体的消息: n bytes

## ack 机制

- 0：生产者不会等待 broker 的 ack，延迟最低，可能丢数据
- 1：服务端会等待 ack 值 leader 副本确认接收到消息后发送 ack，leader切换可能丢数据
- -1：等所有的 follower 的副本收到数据后，leader 发出的 ack，数据不会丢失

## 消费者如何消费数据

KafkaConsumer 类不是线程安全的 （thread-safe），不能在多个线程中共享同一个 KafkaConsumer 实例，否则程序会抛出 `ConcurrentModificationException`异常

1.消费者程序启动多个线程，每个线程维护专属的 KafkaConsumer 实例，负责完整的消息获取、消息处理流程

2.消费者程序使用单或多线程获取消息，同时创建多个消费线程执行消息处理逻辑。