---
title: mysql存储引擎
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-11 09:30:57
password:
summary:
tags:
- mysql
categories:
- mysql
---

## innodb和myisiam区别

**Innodb引擎：**支持数据库ACID事务。并且还提供了行级锁和外键。它的设计的目标就是处理大数据容量的数据库系统，InnoDB 适合频繁修改以及涉及到安全性较高的应用；。

**MyIASM引擎**：不提供事务的支持，也不支持行级锁和外键。MyISAM 适合查询以及插入为主的应用

|                 | **MyISAM**                                                   | **Innodb**                                                   |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存储结构        | **每张表被存放在三个文件：frm表格定义、MYD数据文件、MYI索引文件** | **所有的表都保存在同一个数据文件中**（也可能是多个文件，或者是独立的表空间文件） |
| 存储空间        | MyISAM索引是有压缩的，存储空间较小                           | InnoDB的表需要更多的内存和存储，它会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引 |
| 文件格式        | **数据和索引是分别存储的，数据.MYD，索引.MYI**               | **数据和索引是集中存储的，.ibd**                             |
| 记录存储顺序    | 按记录插入顺序保存                                           | 按主键大小有序插入                                           |
| 外键            | 不支持                                                       | 支持                                                         |
| 事务            | 不支持                                                       | 支持                                                         |
| 锁支持          | 表级锁定                                                     | 行级锁定、表级锁定，锁定力度小并发能力高                     |
| select count(*) | myisam更快，myisam内部维护了计数器。                         |                                                              |
| 索引的实现方式  | B+树索引                                                     | B+树索引                                                     |

### MyISAM索引与InnoDB索引的区别

InnoDB索引是**聚簇索引**，MyISAM索引是**非聚簇索引**。

InnoDB的主键索引的叶子节点存储着行数据，因此主键索引非常高效；MyISAM索引的叶子节点存储的是行数据地址，需要再寻址一次才能得到数据。

InnoDB非主键索引的叶子节点存储的是主键和其他带索引的列数据，因此查询时做到覆盖索引会非常高效

**其他**

- **InnoDB支持事务，MyISAM不支持**
- **InnoDB支持外键，MyISAM不支持**
- **InnoDB支持行锁**（某些情况下还是锁整表，如 `update table set a=1 where user like '%lee%'`
- MyISAM适合查询以及插入为主的应用，InnoDB适合频繁修改以及涉及到安全性较高的应用
- InnoDB中不保存表的行数，如`select count(*) from table`时，InnoDB需要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当`count(*)`语句包含where条件时MyISAM也需要扫描整个表
- 对于自增长的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中可以和其他字段一起建立联合索引
- 清空整个表时，InnoDB是一行一行的删除，效率非常慢。MyISAM则会重建表

## InnoDB引擎的4大特性

### 写缓冲（change buffer）

Insert Buffer 用于非聚集索引的插入和更新操作。先判断插入的非聚集索引是否在缓存池中，如果在则直接插入，否则插入到 Insert Buffer 对象里。再以一定的频率进行 Insert Buffer 和辅助索引叶子节点的 merge 操作，将多次插入合并到一个操作中，减少随机IO带来性能损耗，提高对非聚集索引的插入性能。

> ```text
> 使用 Insert Buffer的两个条件
> - 索引是辅助索引
> - 索引不唯一，如果索引唯一还要去查找索引页进行检查唯一性，就失去了 Insert Buffer 离散插入的性能
> ```

### 二次写

mysql最小的io单位是16k，文件系统io最小的单位是4k，因此存在IO写入导致page损坏的风险

如果数据库发生宕机时，可以通过重做日志对该页进行恢复，但是如果该页本身已经损坏了，进行重做恢复是没有意义的。因此引入了"二次写"方案，解决部分写失败，提高数据页的稳定性。

**流程**

- 对缓存池的脏页进行刷新时，不直接写入磁盘，而是通过 memcpy 函数将脏页复制到内存中的 doublewrite buffer。
- 将内存中的 doublewrite buffer 写入共享表空间的物理磁盘上（备份）。
- 将 doublewrite buffer 中的数据真正的刷新到表磁盘中。
- 如果写 doublewrite buffer 失败，那么这些数据不会写到磁盘，innodb 会载入磁盘原始数据和 redo 日志比较，并重新刷到 doublewrite buffer。
- 如果写 doublewrite buffer 成功，但是刷新到磁盘失败，那么 innodb 就不会通过redo日志来恢复了，而是直接刷新 double write buffer 中的数据到磁盘。

**doublewirte的崩溃恢复**

如果操作系统在将页写入磁盘的过程中发生崩溃，在恢复过程中，innodb存储引擎可以从共享表的空间的doblewrite中找到该页的一个最近副本，将其复制到表空间文件，在应用redo log,就完成了恢复过程。

**副作用：double write带来的写负担**

double write是在物理文件上的一个buffer,会导致系统有更多的fsync操作,而磁盘的fsync性能是很慢的,所以会降低mysql的整体性能

但是doublewrite buffer写入磁盘共享表达空间这个过程是连续存储,是顺序写,性能非常高,牺牲这一点来保证该数据也的完整性还是很有必要的

**关闭double write适合的场景**

- 海量的增删改
- 不惧怕系统数据损坏和丢失
- 系统写负载为主要负载

### 自适应哈希索引

InnoDB 会监控对表上各个索引页的查询，如果观察到通过哈希索引可以带来性能提升，则自动建立哈希索引。自适应哈希索引通过缓存池的 B+ 树页构造而来，因此建立速度很快。

#### 特点

- 无序,没有树高
- 降低对二级索引树的频繁访问资源
- 自适应

#### 缺陷

- hash自适应索引会占用innodb buffer pool;
- 自适应hash索引值适合搜索等值查询,如select * from table where index_col='xxx'，而对于其他查找类型，如范围查找，是不能使用的；
- 极端情况下,自适应hash索引才有比较大的意义

### 预读

数据库访问通常都遵循集中读取原则，使用一些数据大概率会使用附近的数据，这就是所谓的局部性原理，它表明提前加载是有效的，能减少磁盘的i/o。

预读机制就是发起一个i/o请求，异步的在缓冲池中预先回迁若干个页面,预计将会用到的页面回迁.

## 存储引擎选择

如果没有特别的需求，使用默认的Innodb即可。

MyISAM：以读写插入为主的应用程序，比如博客系统、新闻门户网站。

Innodb：更新（删除）操作频率也高，或者要保证数据的完整性；并发量高，支持事务和外键。比如OA自动化办公系统。

## 参考

https://zhuanlan.zhihu.com/p/109528131

https://zhuanlan.zhihu.com/p/64180357

https://www.nasuiyile.cn/219.html

https://www.cnblogs.com/geaozhang/p/7241744.html