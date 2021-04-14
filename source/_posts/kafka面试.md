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
- kafka
- interview
categories:
- kafka
- interview
---



## Kafka 是什么？主要应用场景有哪些？

Kafka 是一个分布式流式处理平台，可以作为企业级的消息引擎。

流平台具有三个关键功能：

1. **消息队列**：发布和订阅消息流。
2. **容错的持久方式存储记录消息流**： Kafka 会把消息持久化到磁盘，有效避免了消息丢失的风险·。
3. **流式处理平台：** 在消息发布的时候进行处理，Kafka 提供了一个完整的流式处理类库。

Kafka 主要有两大应用场景：

1. **消息队列** ：建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。
2. **数据处理：** 构建实时的流数据处理程序来转换或处理数据流。

## Kafka的优势在哪里？

我们现在经常提到 Kafka 的时候就已经默认它是一个非常优秀的消息队列了，我们也会经常拿它给 RocketMQ、RabbitMQ 对比。我觉得 Kafka 相比其他消息队列主要的优势如下：

1. **极致的性能** ：基于 Scala 和 Java 语言开发，设计中大量使用了批量处理和异步的思想，最高可以每秒处理千万级别的消息。
2. **生态系统兼容性无可匹敌** ：Kafka 与周边生态系统的兼容性是最好的没有之一，尤其在大数据和流计算领域。

## 队列模型了解吗？Kafka 的消息模型知道吗？

#### 队列模型：早期的消息模型

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/队列模型23.png)

**使用队列（Queue）作为消息通信载体，满足生产者与消费者模式，一条消息只能被一个消费者使用，未被消费的消息在队列中保留直到被消费或超时。** 比如：我们生产者发送 100 条消息的话，两个消费者来消费一般情况下两个消费者会按照消息发送的顺序各自消费一半（也就是你一个我一个的消费。）

**队列模型存在的问题：**

假如我们存在这样一种情况：我们需要将生产者产生的消息分发给多个消费者，并且每个消费者都能接收到完成的消息内容。

这种情况，队列模型就不好解决了。很多比较杠精的人就说：我们可以为每个消费者创建一个单独的队列，让生产者发送多份。这是一种非常愚蠢的做法，浪费资源不说，还违背了使用消息队列的目的。

#### 发布-订阅模型:Kafka 消息模型

发布-订阅模型主要是为了解决队列模型存在的问题。

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/广播模型21312.png)

发布订阅模型（Pub-Sub） 使用**主题（Topic）** 作为消息通信载体，类似于**广播模式**；发布者发布一条消息，该消息通过主题传递给所有的订阅者，**在一条消息广播之后才订阅的用户则是收不到该条消息的**。

**在发布 - 订阅模型中，如果只有一个订阅者，那它和队列模型就基本是一样的了。所以说，发布 - 订阅模型在功能层面上是可以兼容队列模型的。**

## 什么是Producer、Consumer、Broker、Topic、Partition、Record？

Kafka 将生产者发布的消息发送到 **Topic（主题）** 中，需要这些消息的消费者可以订阅这些 **Topic（主题）**，如下图所示：

![Kafka Topic Partition](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/KafkaTopicPartitioning.png)

上面这张图也为我们引出了，Kafka 比较重要的几个概念：

1. **Producer（生产者）** : 产生消息的一方。
2. **Consumer（消费者）** : 消费消息的一方。
3. **Broker（代理）** : 可以看作是一个独立的 Kafka 实例。多个 Kafka Broker 组成一个 Kafka Cluster。

每个 Broker 中又包含了 Topic 以及 Partition 这两个重要的概念：

- **Topic（主题）** : kafka 通过不同的主题却分不同的业务类型的消息记录。Producer 将消息发送到特定的主题，Consumer 通过订阅特定的 Topic(主题) 来消费消息。
- **Partition（分区）** : Partition 属于 Topic 的一部分。一个 Topic 可以有多个 Partition ，并且同一 Topic 下的 Partition 可以分布在不同的 Broker 上，这也就表明一个 Topic 可以横跨多个 Broker 。
- **记录(Record)?**实际写入到kafka集群并且可以被消费者读取的数据。每条记录包含一个键、值和时间戳。

## 什么是消费者组？

**消费者组是Kafka提供的可扩展且具有容错性的消费者机制。**

Kafka允许你将同一份消息广播到多个消费者组里，以此来丰富多种数据使用场景。

一个消费者组中可以包含多个消费者进程，他们共同消费该topic的数据。有助于消费能力的动态调整；

同一个组下的每个实例都配置有相同的组ID，被分配不同的订阅分区。当某个实例挂掉的时候，其他实例会自动地承担起它负责消费的分区。 因此，消费者组在**一定程度上也保证了消费者程序的高可用性**。

## LEO、LSO、AR、ISR、HW都表示什么含义？

- LEO（Log End Offset）：日志末端位移值或末端偏移量，表示日志下一条待插入消息的位移值。举个例子，如果日志有10条消息，位移值从0开始，那么，第10条消息的位移值就是9。此时，LEO = 10。
- LSO（Log Stable Offset）：这是Kafka事务的概念。如果你没有使用到事务，那么这个值不存在（其实也不是不存在，只是设置成一个无意义的值）。该值控制了事务型消费者能够看到的消息范围。它经常与Log Start Offset，即日志起始位移值相混淆，因为有些人将后者缩写成LSO，这是不对的。
- AR（Assigned Replicas）：AR是主题被创建后，分区创建时被分配的副本集合，副本个数由副本因子决定。
- ISR（In-Sync Replicas）：Kafka中特别重要的概念，指代的是AR中那些与Leader保持同步的副本集合。在AR中的副本可能不在ISR中，但Leader副本天然就包含在ISR中。
- HW（High watermark）：高水位值，这是控制消费者可读取消息范围的重要字段。一个普通消费者只能“看到”Leader副本上介于Log Start Offset和HW（不含）之间的所有消息。水位以上的消息是对消费者不可见的。

## Kafka中位移（offset）的作用

在Kafka中，每个主题分区下的每条消息都被赋予了一个唯一的ID数值，用于标识它在分区中的位置。这个ID数值，就被称为位移，或者叫偏移量。一旦消息被写入到分区日志，它的位移值将不能被修改。

## 磁盘容量规划需要考虑到几个因素？

  - 新增消息数
  - 消息留存时间
  - 平均消息大小
  - 备份数
  - 是否启用压缩

## __consumer_offsets 是做什么用的？

这是一个内部主题，主要用于存储消费者的偏移量，以及消费者的元数据信息（消费者实例，消费者id等等）

需要注意的是：Kafka的GroupCoordinator组件提供对该主题完整的管理功能，包括该主题的创建、写入、读取和Leader维护等。

## Kafka 的多副本机制了解吗？带来了什么好处？

 Kafka 为分区（Partition）引入了多副本（Replica）机制。Kafka副本当前分为领导者副本和追随者副本。只有Leader副本才能对外提供读写服务，响应Clients端的请求。Follower副本只是采用拉（PULL）的方式，被动地同步Leader副本中的数据，并且在Leader副本所在的Broker宕机后，随时准备应聘Leader副本。

**Kafka 的多分区（Partition）以及多副本（Replica）机制有什么好处呢？**

1. Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力（负载均衡）。
2. 多副本提高了消息存储的安全性, 提高了容灾能力，不过也相应的增加了所需要的存储空间。

## Zookeeper 在 Kafka 中的作用知道吗？

目前，Kafka使用ZooKeeper存放集群元数据、成员管理、Controller选举，以及其他一些管理类任务。

- 存放元数据”是指主题分区的所有数据都保存在 ZooKeeper 中，且以它保存的数据为权威，其他 “人” 都要与它保持对齐。
- “成员管理” 是指 Broker 节点的注册、注销以及属性变更，等等。
- “Controller 选举” 是指选举集群 Controller，而其他管理类任务包括但不限于主题删除、参数配置等。

##  Kafka 如何保证消息的消费顺序？

Partition(分区)是真正保存消息的地方。特定 Topic 可以指定多个 Partition。

Kafka 只能为我们保证 Partition(分区) 中的消息有序，而不能保证 Topic(主题) 中的消息有序。

如果需要保证主题消息的顺序性，可以**1 个 Topic 只对应一个 Partition**。这样当然可以解决问题，但是破坏了 Kafka 的设计初衷。

Kafka 中发送 1 条消息的时候，可以指定 topic, partition, key,data（数据） 4 个参数。类似的，如果你发送消息的时候指定了 Partition 的话，所有消息都会被发送到指定的 Partition。并且，同一个 key 的消息可以保证只发送到同一个 partition，这个我们可以采用表/对象的 id 来作为 key 。

总结一下，对于如何保证 Kafka 中消息消费的顺序，有了下面两种方法：

1. **1 个 Topic 只对应一个 Partition**。
2. （推荐）**发送消息的时候指定 key/Partition。**

当然不仅仅只有上面两种方法，上面两种方法是我觉得比较好理解的，

## Kafka 如何保证消息不丢失

### 生产者丢失消息的情况

生产者(Producer) 调用`send`方法发送消息之后，消息可能因为网络问题并没有发送过去。 

所以，我们不能默认在调用`send`方法发送消息之后消息消息发送成功了。为了确定消息是发送成功，我们要判断消息发送的结果。可以采用为其添加回调函数的形式如果消息发送失败的话，我们检查失败的原因之后重新发送即可。

**另外这里推荐为 Producer 的`retries `（重试次数）设置一个比较合理的值，一般是 3 ，但是为了保证消息不丢失的话一般会设置比较大一点。设置完成之后，当出现网络问题之后能够自动重试消息发送，避免消息丢失。**

### 消费者丢失消息的情况

当消费者拉取到了分区的某个消息之后，消费者会自动提交了 offset。试想一下，当消费者刚拿到这个消息准备进行真正消费的时候，突然挂掉了，消息实际上并没有被消费，但是 offset 却被自动提交了。

**解决办法关闭自动提交 offset，每次在真正消费完消息之后之后再自己手动提交 offset 。**这样会带来消息被重新消费的问题。比如你刚刚消费完消息之后，还没提交 offset，结果自己挂掉了，那么这个消息理论上就会被消费两次。

### Kafka 弄丢了消息

 我们知道 Kafka 为分区（Partition）引入了多副本（Replica）机制。分区（Partition）中的多个副本之间会有一个叫做 leader 的家伙，其他副本称为 follower。我们发送的消息会被发送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步。生产者和消费者只与 leader 副本交互。你可以理解为其他副本只是 leader 副本的拷贝，它们的存在只是为了保证消息存储的安全性。

**试想一种情况：假如 leader 副本所在的 broker 突然挂掉，那么就要从 follower 副本重新选出一个 leader ，但是 leader 的数据还有一些没有被 follower 副本的同步的话，就会造成消息丢失。**

**设置 acks = all**

解决办法就是我们设置  **acks = all**。acks 是 Kafka 生产者(Producer)  很重要的一个参数。

acks 的默认值即为1，代表我们的消息被leader副本接收之后就算被成功发送。当我们配置 **acks = all** 代表则所有副本都要接收到该消息之后该消息才算真正成功被发送。

**设置 replication.factor >= 3**

为了保证 leader 副本能有 follower 副本能同步消息，我们一般会为 topic 设置 **replication.factor >= 3**。这样就可以保证每个 分区(partition) 至少有 3 个副本。虽然造成了数据冗余，但是带来了数据的安全性。

**设置 min.insync.replicas > 1**

一般情况下我们还需要设置 **min.insync.replicas> 1** ，这样配置代表消息至少要被写入到 2 个副本才算是被成功发送。**min.insync.replicas** 的默认值为 1 ，在实际生产中应尽量避免默认值 1。

但是，为了保证整个 Kafka 服务的高可用性，你需要确保 **replication.factor > min.insync.replicas** 。为什么呢？设想一下假如两者相等的话，只要是有一个副本挂掉，整个分区就无法正常工作了。这明显违反高可用性！一般推荐设置成 **replication.factor = min.insync.replicas + 1**。

**设置 unclean.leader.election.enable = false**

我们最开始也说了我们发送的消息会被发送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步。多个 follower 副本之间的消息同步情况不一样，当我们配置了 **unclean.leader.election.enable = false**  的话，当 leader 副本发生故障时就不会从  follower 副本中和 leader 同步程度达不到要求的副本中选择出  leader ，这样降低了消息丢失的可能性。

## Kafka 如何保证消息不重复消费

### ？？？

## **Kafka为什么性能好**

### **顺序写**

操作系统每次从磁盘读写数据的时候，需要先寻址，也就是先要找到数据在磁盘上的物理位置，然后再进行数据读写，如果是机械硬盘，寻址就需要较长的时间。 Kafka 用的是顺序写，追加数据是追加到末尾，磁盘顺序写的性能极高。

### **零拷贝**

Kafka 利用了 Linux 的 sendFile 技术实现零拷贝，减少了内核和用户模式之间的上下文切换。

零拷贝是指将数据直接从磁盘文件复制到网卡设备中。

### **Kafka 的网络设计**

加强版的 Reactor 网络线程模型

### Batching of Messages

批量量处理。合并小的请求，然后以流的方式进行交互，直顶网络上限。

## Kafka为什么不支持读写分离？

CAP理论下，我们只能保证C（可用性）和A（一致性）取其一，如果支持读写分离，那其实对于一致性的要求可能就会有一定折扣，因为通常的场景下，副本之间都是通过同步来实现副本数据一致的，那同步过程中肯定会有时间的消耗，如果支持了读写分离，就意味着可能的数据不一致，或数据滞后。

## Controller发生网络分区（Network Partitioning）时，Kafka会怎么样？

一旦发生Controller网络分区，那么，第一要务就是查看集群是否出现“脑裂”，即同时出现两个甚至是多个Controller组件。这可以根据Broker端监控指标ActiveControllerCount来判断。

由于Controller会给Broker发送3类请求，LeaderAndIsrRequest，StopReplicaRequest，UpdateMetadataRequest，因此，一旦出现网络分区，这些请求将不能顺利到达Broker端。

这将影响主题的创建、修改、删除操作的信息同步，表现为集群仿佛僵住了一样，无法感知到后面的所有操作。

## Java Consumer 为什么采用单线程来获取消息？

Java Consumer是双线程的设计。一个线程是用户主线程，负责获取消息；另一个线程是心跳线程，负责向Kafka汇报消费者存活情况。将心跳单独放入专属的线程，能够有效地规避因消息处理速度慢而被视为下线的“假死”情况。

单线程获取消息的设计能够避免阻塞式的消息获取方式。单线程轮询方式容易实现异步非阻塞式，这样便于将消费者扩展成支持实时流处理的操作算子。因为很多实时流处理操作算子都不能是阻塞式的。另外一个可能的好处是，可以简化代码的开发。多线程交互的代码是非常容易出错的。

## 简述Follower副本消息同步的完整流程

首先，Follower发送FETCH请求给Leader。

接着，Leader会读取底层日志文件中的消息数据，再更新它内存中的Follower副本的LEO值，更新为FETCH请求中的fetchOffset值。

最后，尝试更新分区高水位值。

Follower接收到FETCH响应之后，会把消息写入到底层日志，接着更新LEO和HW值。

Leader和Follower的HW值更新时机是不同的，Follower的HW更新永远落后于Leader的HW。这种时间上的错配是造成各种不一致的原因。

## Leader总是-1，怎么破？

通常情况下就是Controller不工作了，导致无法分配leader。

删除ZooKeeper节点/controller，触发Controller重选举。Controller重选举能够为所有主题分区重刷分区状态，可以有效解决因不一致导致的 Leader 不可用问题。

或者重启Controller节点上的Kafka进程，让其他节点重新注册Controller角色。

## 如何设置Kafka能接收的最大消息的大小？

- Broker端参数：message.max.bytes，max.message.bytes（topic级别），replica.fetch.max.bytes（否则follower会同步失败）
- Consumer端参数：fetch.message.max.bytes

## Kafka能手动删除消息吗？

Kafka不需要用户手动删除消息。它本身提供了留存策略，能够自动删除过期消息。当然，它是支持手动删除消息的。

- 对于设置了Key且参数cleanup.policy=compact的主题而言，我们可以构造一条 的消息发送给Broker，依靠Log Cleaner组件提供的功能删除掉该 Key 的消息。
- 对于普通主题而言，我们可以使用kafka-delete-records命令，或编写程序调用Admin.deleteRecords方法来删除消息。这两种方法殊途同归，底层都是调用Admin的deleteRecords方法，通过将分区Log Start Offset值抬高的方式间接删除消息。

## 如何确定合适的Kafka主题的分区数量？

选择合适的分区数量可以达到高度并行读写和负载均衡的目的，在分区上达到均衡负载是实现吞吐量的关键。需要根据每个分区的生产者和消费者的期望吞吐量进行估计。

举个栗子：假设期望读取数据的速率(吞吐量)为**1GB/Sec**，而一个消费者的读取速率为**50MB/Sec**，此时至少需要20个分区以及20个消费者(一个消费者组)。同理，如果期望生产数据的速率为**1GB/Sec**，而每个生产者的生产速率为**100MB/Sec**，此时就需要有10个分区。在这种情况下，如果设置20个分区，既可以保障**1GB/Sec**的生产速率，也可以保障消费者的吞吐量。通常需要将分区的数量调整为消费者或者生产者的数量，只有这样才可以同时实现生产者和消费者的吞吐量。

## kafka如何实现延迟队列？

Kafka并没有使用JDK自带的Timer或者DelayQueue来实现延迟的功能，而是**基于时间轮自定义了一个用于实现延迟功能的定时器（SystemTimer）**。JDK的Timer和DelayQueue插入和删除操作的平均时间复杂度为`O(nlog(n))`，并不能满足Kafka的高性能要求，而基于时间轮可以将插入和删除操作的时间复杂度都降为`O(1)`。

**底层使用数组实现，**数组中的每个元素可以存放一个TimerTaskList对象。TimerTaskList是一个环形双向链表，在其中的链表项TimerTaskEntry中封装了真正的定时任务TimerTask.

**推进时间？**Kafka中的定时器借助了JDK中的DelayQueue来协助推进时间轮。具体做法是对于每个使用到的TimerTaskList都会加入到DelayQueue中。Kafka中的`TimingWheel`专门用来执行插入和删除TimerTaskEntry的操作，而DelayQueue专门负责时间推进的任务。再试想一下，DelayQueue中的第一个超时任务列表的expiration为200ms，第二个超时任务为840ms，这里获取DelayQueue的队头只需要O(1)的时间复杂度。如果采用每秒定时推进，那么获取到第一个超时的任务列表时执行的200次推进中有199次属于“空推进”，而获取到第二个超时任务时有需要执行639次“空推进”，这样会无故空耗机器的性能资源，**这里采用DelayQueue来辅助以少量空间换时间，从而做到了“精准推进”**。Kafka中的定时器真可谓是“知人善用”，用TimingWheel做最擅长的任务添加和删除操作，而用DelayQueue做最擅长的时间推进工作，相辅相成。

## **Kafka 判断一个节点是否还活着有那两个条件？**

（1）节点必须可以维护和 ZooKeeper 的连接，Zookeeper 通过心跳机制检查每个节点的连接

（2）如果节点是个 follower,他必须能及时的同步 leader 的写操作，延时不能太久

## **Kafka 消息是采用 Pull 模式，还是 Push 模式？**

producer 将消息推送到 broker，consumer 从 broker 拉取消息.

**Kafka 存储在硬盘上的消息格式是什么？**

消息由一个固定长度的头部和可变长度的字节数组组成。头部包含了一个版本号和 CRC32校验码。

- 消息长度: 4 bytes (value: 1+4+n)
- 版本号: 1 byte
- CRC 校验码: 4 bytes
- 具体的消息: n bytes

## **kafka 的 ack 机制**

request.required.acks 有三个值 0 1 -1

0:生产者不会等待 broker 的 ack，这个延迟最低但是存储的保证最弱当 server 挂掉的时候就会丢数据

1：服务端会等待 ack 值 leader 副本确认接收到消息后发送 ack 但是如果 leader 挂掉后他不确保是否复制完成新 leader 也会导致数据丢失

-1：同样在 1 的基础上 服务端会等所有的 follower 的副本受到数据后才会受到 leader 发出的 ack，这样数据不会丢失

## Kafka 的消费者如何消费数据

消费者每次消费数据的时候，消费者都会记录消费的物理偏移量（offset）的位置

等到下次消费时，他会接着上次位置继续消费