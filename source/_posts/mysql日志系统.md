---
title: mysql日志系统
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-11 09:04:56
password:
summary:
tags:
- mysql
categories:
- mysql
---

##  日志系统

### redo log（重做日志）

1. InnoDB引擎的日志，redo log 保证数据库异常重启之前提交的记录不会丢失（crash-safe）

2. 在一条更新语句进行执行的时候，InnoDB引擎会把更新记录写到 redo log 日志中，然后更新内存，此时算是语句执行完了，然后在空闲的时候或者是按照设定的更新策略将 redo log 中的内容更新到磁盘中，这里涉及到WAL即Write Ahead logging技术（先写日志再写磁盘）

3. redo log 是物理日志，记录的是在某个数据页上做了什么修改

4. redo log是循环写，空间固定会用完

### binlog（归档日志）

1. server层日志，记录了MySQL对数据库执行更改的所有操作，没有crash-safe能力

2. binlog是逻辑日志，记录的是记录所有数据库表结构变更（例如CREATE、ALTER、TABLE…）以及表数据修改（INSERT、UPDATE、DELETE…）的二进制日志

3. binlog采用追加写的模式

4. **用途：**  

  (1)恢复：binlog日志恢复数据库数据  

  (2)复制：主库有一个log dump线程，将binlog传给从库，从库有两个线程，一个I/O线程，一个SQL线程，I/O线程读取主库传过来的binlog内容并写入到relay log，SQL线程从relay log里面读取内容，写入从库的数据库  

  (3)审计：用户可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入攻击

**binlog常见格式**

| format    | 定义                       | 优点                           | 缺点                                                         |
| --------- | -------------------------- | ------------------------------ | ------------------------------------------------------------ |
| statement | 记录的是修改SQL语句        | 日志文件小，节约IO，提高性能   | 确性差，有些语句的执行结果是依赖于上下文命令可能会导致主备不一致（delete带limit，很可能会出现主备数据不一致的情况） |
| row       | 记录的是每行实际数据的变更 | 准确性强，能准确复制数据的变更 | 日志文件大，较大的网络IO和磁盘IO                             |
| mixed     | statement和row模式的混合   | 准确性强，文件大小适中         | 有可能发生主从不一致问题                                     |

### 两段提交  

1. 两段提交保证数据库binlog状态和日志redo log恢复出来的数据库状态保持一致

2. 两段提交: 写入redo log处于prepare阶段 --（A）- 写入bin log -（B）-- 提交事务处于commit状态 

  (1)时刻A崩溃恢复: redo log未提交， bin log 未写，不会传到备库，这时事务会回滚  

  (2)时刻B崩溃恢复: 如果 redo log 事务完整有commit标识则直接提交，如果 redo log 事务只有完整的prepare，则判断对应事务 bin log 是否完整，是提交事务，否则回滚事务

3. bin log完整性判断:  

  (1)statement格式最后有commit  

  (2)row格式最有有一个XID event（redo log 和 bin log关联：有一个共同字段XID）

### undo log

1. undo log是逻辑日志，可以认为当delete一条记录时，undo log中会记录一条对应的insert记录，反之亦然，当update一条记录时，它记录一条对应相反的update记录

2. undo log 作用  

  (1)提供回滚  

  (2)多个行版本控制（MVCC）