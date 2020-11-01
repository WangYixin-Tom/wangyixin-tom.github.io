---
title: redis主从同步
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-25 16:30:01
password:
summary:
tags:
- redis
categories:
- redis
---

[toc]

主从库之间采用读写分离。
读操作：主库、从库都可以接收；
写操作：首先到主库执行，然后，主库将写操作同步给从库。
![读写分离](duxiefenli.jpg)

## 同步机制

通过 `replicaof`（Redis 5.0 之前使用 `slaveof`）命令形成主库和从库的关系。

1、主从库间建立连接、协商同步，为全量复制做准备。
**从库和主库建立起连接，发送 psync 命令**，表示要进行数据同步，**主库确认回复**，FULLRESYNC响应表示第一次复制采用的全量复制。
psync 命令包含了主库的 runID 和复制进度 offset 两个参数。

2、主库将所有数据同步给从库。从库收到数据后，在本地完成数据加载。
主库执行 bgsave 命令，生成 RDB 文件，接着将文件发给从库。
从库接收到 RDB 文件后，会先清空当前数据库（避免之前数据的影响），然后加载 RDB 文件。

3、主库会把第二阶段执行过程中新收到的写命令(replication buffer中的修改操作)，再发送给从库。
主库会在内存中使用 replication buffer，记录 RDB 文件生成后收到的所有写操作。

![主从第一次同步](zhucongtongbu.jpg)

## 主从级联

### 问题

- 从库数量很多且都要和主库进行全量复制时，会导致主库忙于 fork 子进程生成 RDB 文件，进行数据全量同步。fork 会阻塞主线程处理正常请求，从而导致主库响应应用程序的请求速度变慢。

- 传输 RDB 文件也会占用主库的网络带宽，同样会给主库的资源使用带来压力。

### 解决

通过“主 - 从 - 从”模式将主库生成 RDB 和传输 RDB 的压力，以级联的方式分散到从库上

![级联的“主-从-从”模式](master_slave_slave.jpg)



## 网络闪断

网络闪断后，主从库会采用增量复制的方式继续同步。

### 增量复制机制

- 主库把断连期间收到的写操作命令写入 repl_backlog_buffer 缓冲区

- repl_backlog_buffer 是一个环形缓冲区，主库会记录写到的位置，从库会记录已经读到的位置。
- 连接恢复后，从库给主库发送 psync 命令，并把自己当前的 slave_repl_offset 发给主库，主库会判断 master_repl_offset 和 slave_repl_offset 之间的差距。把 master_repl_offset 和 slave_repl_offset 之间的命令操作同步给从库。
- 库还未读取的操作被主库新写的操作覆盖，需要全量复制

### 应对

- repl_backlog_size 设置一个合理的值

- 使用切片集群来分担单个主库的请求压力