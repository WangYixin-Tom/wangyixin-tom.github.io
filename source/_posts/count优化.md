---
title: count优化
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-13 18:30:14
password:
summary:
tags:
- mysql
categories:
- mysql
---

## count说明

### count(a) 和 count(*)

当 count() 统计某一列时，比如 count(a)，a 表示列名，是不统计 null 的。

而 count(*) 无论是否包含空值，都会统计。

### MyISAM 引擎和 InnoDB 引擎 count(*) 的区别

对于 MyISAM 引擎，如果没有 where 子句，也没检索其它列，那么 count(*) 将会非常快。因为 MyISAM 引擎会把
表的总行数存在磁盘上。

InnoDB 并不会保留表中的行数，因为并发事务可能同时读取到不同的行数。所以执行 count(*) 时都是临时去
计算的，会比 MyISAM 引擎慢很多。

### MySQL 5.7.18 前后 count(*) 的区别

MySQL 5.7.18 之前，InnoDB 通过扫描聚簇索引来处理 count(*) 语句。

MySQL 5.7.18 开始，通过遍历最小的可用二级索引来处理 count(*) 语句。

优化器基于成本的考虑，优先选择的是二级索引。所以 count(主键) 其实没 count (*)快。

### count(1)   count(*)

执行计划相同，速度没有明显差别

## count 优化

1、show table status,Rows 这列就表示这张表的行数。

```mysql
show table status like 't1';
```

估算值，可能与实际值相差 40% 到 50%。

2、用 Redis 做计数器

redis 和数据库访问存在时间先后，可能会读到错误的值。

3、增加计数表

维持了计数的准确性

