---
title: MVCC
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-14 20:14:27
password:
summary:
tags:
- mysql
categories:
- mysql
---

## MVCC

多版本并发控制。

> MVCC 只在 RC 和 RR 两个隔离级别下工作。
>
> 不管需要执行多长时间，每个事务看到的数据都是一致的。根据事务开始的时间不同，每个事务对同一张表，同一时刻看到的数据可能是不一样的。

## Undo log

undo log 是逻辑日志，将数据库逻辑地恢复到原来的样子，所有修改都被逻辑地取消了。

undo log 的另一个作用是 MVCC， MVCC 的实现是通过 undo 来完成的。当用户读取一行记录时，可以通过 undo log 读取之前的行版本信息，以此实现非锁定读取。

## MVCC 的实现原理

通过保存数据在某个时间点的快照来实现的：

InnoDB 每一行数据都有一个隐藏的回滚指针，用于指向该行修改前的最后一个历史版本（存放在 UNDO LOG中）。

## MVCC 的优势

MVCC 最大的好处是读不加锁，读写不冲突，极大地增加了 MySQL 的并发性。
通过 MVCC，保证了事务 ACID 中的隔离性特性。