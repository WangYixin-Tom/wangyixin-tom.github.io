---
title: join优化
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-13 17:35:27
password:
summary:
tags:
- mysql
categories:
- mysql
---

## 关联查询

### Nested-Loop Join

**思想**

一次一行循环：从驱动表中读取行并取到关联字段，根据关联字段在被驱动表取出满足条件的行（使用索引），然后取两张表的结果合集。[manual](https://dev.mysql.com/doc/refman/5.7/en/nested-loop-joins.html)

> 在关联字段有索引时，才会使用 NLJ，如果没索引，就会使用 Block Nested-Loop Join。
>
> 一般 join 语句中，没有出现Using join buffer就表示使用的join 算法是 NLJ

### Block Nested-Loop Join 

**思想**：把驱动表的数据读入到 join_buffer 中，然后扫描被驱动表，把被驱动表每一行取出来跟 join_buffer 中的数据做对比，如果满足 join 条件，则返回结果给客户端。[manual](https://dev.mysql.com/doc/refman/5.7/en/bnl-bkaoptimization.html)

> join_buffer减少了磁盘扫描次数。

**主要影响**

1. 可能会多次扫描被驱动表，占用磁盘IO资源；
2. 判断join条件需要执行M*N次对比（M、N分别是两张表的行数），如果是大表就会占用非常多的CPU资源；
3. 可能会导致Buffer Pool的热数据被淘汰，影响内存命中率。

### Batched Key Access

> **MRR**
>
> 尽量使用顺序读盘。如果按照主键的递增顺序查询的话，对磁盘的读比较接近顺序读，能够提升读性能。
>
> MRR能够提升性能的核心在于，查询语句在索引的是范围查询，可以得到足够多的主键id。排序后，再去主键索引查数据，才能体现出“顺序性”的优势。

**思想**

结合NLJ和BNL，将驱动表中相关列放入 join_buffer 中，批量将关联字段的值发送到 Multi-Range Read（MRR） 接口，MRR 通过接收到的值，根据其对应的主键 ID 进行排序，然后再进行数据的读取和操作，返回结果给客户端。

```mysql
set optimizer_switch='mrr=on,mrr_cost_based=off,batched_key_access=on';/*开启BKA*/
```

> Extra显示Using join buffer （Batched Key Access）

## 优化关联查询

### 关联字段添加索引

被驱动表的关联字段上添加索引，让 BNL变成 NLJ 或者 BKA ，可明显优化关联查询。

### 小表做驱动表

复杂度是低于大表做驱动表的。

### 临时表

> 某条关联查询只是临时查一次，再去添加索引会浪费资源。

尝试创建临时表，然后在临时表中的关联字段上添加索引，然后通过临时表来做关联查询。



