---
title: redis分布式锁
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-29 23:52:36
password:
summary:
tags:
- redis
categories:
- redis
---

为了保证并发访问的正确性，Redis 提供了两种方法，分别是加锁和原子操作。

## 原子操作

- 单命令操作（INCR/DECR）；
- 把多个操作写到一个 Lua 脚本中，以原子性方式执行单个 Lua 脚本

## 分布式锁

- 分布式锁的加锁和释放锁的过程，涉及多个操作。需要保证这些锁操作的原子性；

- 共享存储系统保存了锁变量，需要考虑保证共享存储系统的可靠性，进而保证锁的可靠性。

### 单个 Redis 节点的分布式锁

- **SETNX key value**

key 不存在， key 会被创建。执行完业务逻辑后，使用 DEL 命令删除锁变量，从而释放锁。

**可能风险：**

  - 操作共享数据时发生了异常，结果一直没有执行最后的 DEL 命令释放锁。
  - 客户端 A 执行了 SETNX 命令加锁后，客户端 B 执行了 DEL 命令释放锁，客户端 A 的锁就被误释放

+ `SET key value [EX seconds | PX milliseconds]  [NX]`

### 多个 Redis 节点的高可靠分布式锁

分布式锁算法 Redlock

- 客户端获取当前时间。
- 客户端按顺序依次向 N 个 Redis 实例执行加锁操作。
- 一旦客户端完成了和所有 Redis 实例的加锁操作，客户端就要计算整个加锁过程的总耗时

**加锁成功条件**

- 客户端从超过半数（大于等于 N/2+1）的 Redis 实例上成功获取到了锁；

- 客户端获取锁的总耗时没有超过锁的有效时间。