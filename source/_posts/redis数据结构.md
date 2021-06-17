---
title: redis数据结构
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-26 21:44:02
password:
summary:
tags:
- redis
categories:
- redis
---



## 基本数据结构

包括：String（字符串）、List（列表）、Hash（哈希）、Set（集合）和 Sorted Set（有序集合）

| 基本数据结构 | 底层实现           |
| ------------ | ------------------ |
| string       | 动态字符串         |
| List         | 双向链表、压缩列表 |
| Hash         | 哈希表，压缩列表   |
| Sorted Set   | 跳表，压缩列表     |
| Set          | 哈希表、数组       |

redis中的键值对采用哈希表，哈希表就是一个数组，数组的每个元素称为一个哈希桶，每个哈希桶中保存了键值对数据的指针。
哈希冲突采用链式哈希。同一个哈希桶中的多个元素用一个链表来保存，它们之间依次用指针连接。
冲突增多时，Redis 会对哈希表做渐进式 rehash操作。

> rehash 也就是增加现有的哈希桶数量，让逐渐增多的 entry 元素能在更多的桶之间分散保存，减少单个桶中的元素数量，从而减少单个桶中的冲突。渐进式 rehash时指拷贝数据时，Redis 仍然正常处理客户端请求，每处理一个请求时，从哈希表 1 中的第一个索引位置开始，顺带着将这个索引位置上的所有 entries 拷贝到哈希表 2 中；等处理下一个请求时，再顺带拷贝哈希表 1 中的下一个索引位置的 entries

## RedisObject

```c
struct RedisObject {
    int4 type; // 4bits，类型
    int4 encoding; // 4bits，存储形式
    int24 lru; // 24bits，LRU 信息
    int32 refcount; // 4bytes，引用计数
    void *ptr; // 8bytes，对象内容的具体存储位置
} robj;
```

 RedisObject 对象头需要占据 16 字节的存储空间。

## 字符串

### 数据结构

Redis 的字符串叫着「SDS」，也就是Simple Dynamic String。它的结构是一个带长度信息的字节数组。

优点：

1. 相比C语言字符串，使获取字符串长度时间复杂度降为`O(1)`
2. **杜绝缓冲区溢出**：当SDS API需要对SDS进行修改时，API会先检查SDS当前剩余空间是否满足修改之后所需的空间，如果不满足的话API会自动将SDS的空间扩展至修改之后所需空间大小，然后再执行实际的修改操作，所以SDS不会出现缓冲区溢出问题。
3. 减少修改字符串时带来的内存重分配次数： 在SDS中通过未使用空间解除了字符串长度和底层数组长度之间的关联，在SDS中，buf数组长度不一定是字符串长度加1，数组中可能包含未使用的字节，这些字节的数量就是由SDS的free属性记录。通过未使用空间，SDS实现了空间预分配和惰性空间释放两种优化策略。

```c
struct SDS<T> {
    T capacity; // 数组容量
    T len; 		// 数组长度，已经使用的容量
    byte flags; // 特殊标识位，不理睬它
    byte[] content; // 数组内容
}

struct SDS {
    int8 capacity; // 1byte
    int8 len; // 1byte
    int8 flags; // 1byte
    byte[] content; // 内联数组，长度为 capacity
}
```

- embstr ：将 RedisObject 对象头和 SDS 对象连续存在一起，使用 malloc 方法一次分配。
- raw ：需要两次 malloc，两个对象头在内存地址上一般是不连续的。

### 空间预分配策略

- 字符串在长度小于 1M 之前，扩容空间采用加倍策略。
- 当长度超过 1M 之后，为了避免加倍后导致浪费，多分配 1M 大小的冗余空间。

### 惰性空间释放

用于优化SDS的字符串收缩操作，当字符串收缩时，程序不会立即执行内存重分配来回收收缩后内存多出来的空间，而是使用free属性记录下来，以备将来使用。

## List

redis list数据结构底层采用压缩列表ziplist或双向列表两种数据结构进行存储，首先以ziplist进行存储，在不满足ziplist的存储要求后转换为双向列表。

**当列表对象同时满足以下两个条件时，列表对象使用ziplist进行存储，否则用双向列表存储。**

- 列表对象保存的所有字符串元素的长度小于64字节
- 列表对象保存的元素数量小于512个。

## 字典

### 应用

- hash 结构的数据
- 整个 Redis 数据库的所有 key 和 value 组成了一个全局字典。
- 还有带过期时间的 key 集合也是一个字典。
- zset 集合中存储 value 和 score 值的映射关系。

### 数据结构

内部包含两个 hashtable，通常情况下只有一个 hashtable 是有值的。

扩容缩容时，需要分配新的 hashtable，然后进行渐进式搬迁。

两个 hashtable 存储的分别是旧的 hashtable 和新的 hashtable。待搬迁结束后，旧的 hashtable 被删除，新的 hashtable 取而代之。

```c
struct dict {
    ...
    dictht ht[2];		//哈希表
}
struct dictht {
    dictEntry** table; 	// 二维
    long size; 			// 第一维数组的长度
    long used; 			// hash 表中的元素个数
    ...
}
struct dictEntry {
    void* key;
    void* val;
    dictEntry* next; 	// 链接下一个 entry
}
```

### hash表扩容机制

1、redis字典（hash表）底层有两个数组，还有一个rehashidx用来控制rehash

2、初始默认hash长度为4，当元素个数与hash表长度一致时，就发生扩容，hash长度变为原来的二倍

3、**redis中的hash则是执行的单步rehash的过程**：每次的增删改查，rehashidx+1，然后执行对应原hash表rehashidx索引位置的rehash

### 渐进式rehash

大字典的扩容是比较耗时间的，需要重新申请新的数组，然后将旧字典所有链表中的元素重新挂接到新的数组下面，`O(n)`级别的操作。

Redis还会在定时任务中对字典进行主动搬迁。

1. 为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两个哈希表

2. 将rehashindex的值设置为0，表示rehash工作正式开始

3. 在rehash期间，每次对字典执行增删改查操作是，程序除了执行指定的操作以外，还会顺带将ht[0]哈希表在rehashindex索引上的所有键值对rehash到ht[1]，当rehash工作完成以后，rehashindex的值+1

4. 随着字典操作的不断执行，最终会在某一时间段上ht[0]的所有键值对都会被rehash到ht[1]，这时将rehashindex的值设置为-1，表示rehash操作结束

### 扩容条件

当 hash 表中元素的个数等于第一维数组的长度时，就会开始扩容，扩容的新数组是原数组大小的 2 倍。

如果 Redis 正在做 bgsave，为了减少内存页的过多分离 （Copy On Write），Redis 尽量不去扩容 （dict_can_resize）

如果 hash 表已经非常满了，元素的个数已经达到了第一维数组长度的 5 倍 （dict_force_resize_ratio），说明 hash 表已经过于拥挤了，就会强制扩容。

## 压缩列表
压缩列表类似于一个数组，数组中的每一个元素都对应保存一个数据。

不同的是，压缩列表在表头有三个字段 zlbytes、zltail 和 zllen，分别表示列表占用字节数、列表尾的偏移量和列表中的 entry 个数；压缩列表在表尾还有一个 zlend，表示列表结束。

压缩列表是在内存中分配一块地址连续的空间，然后把集合中的元素一个接一个地放在这块空间内，非常紧凑。

```c
struct ziplist<T> {
    int32 zlbytes; // 整个压缩列表占用字节数
    int32 zltail_offset; // 最后一个元素距离压缩列表起始位置的偏移量，用于快速定位到最后一个节点
    int16 zllength; // 元素个数
    T[] entries; // 元素内容列表，挨个挨个紧凑存储
    int8 zlend; // 标志压缩列表的结束，值恒为 0xFF
}
struct entry {
    int<var> prevlen; // 前一个 entry 的字节长度
    int<var> encoding; // 元素类型编码
    optional byte[] content; // 元素内容
}
```

## 跳表
跳表在链表的基础上，增加了多级索引，通过索引位置的几个跳转，实现数据的快速定位。

## 复杂度

各个数据结构的查找时间复杂度

| 数据结构 | 复杂度    |
| -------- | --------- |
| 哈希表   | `O(1)`    |
| 跳表     | `O(logN)` |
| 双向链表 | `O(N)`    |
| 压缩列表 | `O(N)`    |
| 数组     | `O(N)`    |

## Bitmap 

Bitmap 本身是用 String 类型作为底层数据结构实现的一种统计二值状态的数据类型。

可用于用户连续签到，可在记录海量数据时，Bitmap 能够有效地节省内存空间。

## HyperLogLog 

Hyperloglog 提供了一种不太精确的基数统计方法，用来统计一个集合中不重复的元素个数，比如统计网站的UV，或者应用的日活、月活，存在一定的误差。

## GEO

适用于位置信息服务（Location-Based Service，LBS）的应用。

**实现原理**

GEO 类型的底层数据结构就是用 Sorted Set 来实现的。Sorted Set 元素的权重分数是一个浮点数（float 类型），而一组经纬度包含的是经度和纬度两个值。因袭需要对一组经纬度进行 GeoHash 编码，基本原理就是`二分区间，区间编码`，经纬度编码需要交叉组合成一个数。

## Streams（5.0）

Streams 是 Redis 专门为消息队列设计的数据类型。

对于插入的每一条消息，Streams 可以自动为其生成一个全局唯一的 ID。

Streams支持消费组。消息队列中的消息一旦被消费组里的一个消费者读取了，就不能再被该消费组内的其他消费者读取了。

Streams 会自动使用内部队列（也称为 PENDING List）留存消费组里每个消费者读取的消息，直到消费者使用 XACK 命令通知 Streams“消息已经处理完成”