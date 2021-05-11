---
title: order/group by优化
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-13 15:45:38
password:
summary:
tags:
- mysql
categories:
- mysql
---

## order by 原理

按照排序原理分[manual](https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html)，MySQL 排序方式分两种：

- 通过有序索引直接返回有序数据：Using index
- 通过 Filesort 进行的排序：Using filesort

Filesort是内存排序还是磁盘排序取决于：

-  “排序的数据大小” < sort_buffer_size: 内存排序
-  “排序的数据大小” > sort_buffer_size: 磁盘排序

通过trace中的`number_of_tmp_files`，等于 0，则表示排序过程没使用临时文件，在内存中就能完成排序。

**Filesort 下的排序模式**

- < sort_key, rowid >双路排序（回表排序模式）：取出排序字段和行 ID，在 sort buffer 中排序，排序完后再次取回其它需要的字段；
- < sort_key, additional_fields >单路排序：是一次性取出满足条件行的所有字段，然后在sort buffer中进行排序；
- < sort_key, packed_additional_fields >打包数据排序模式：与单路排序相似，区别是将 char 和 varchar 字段存到sort buffer 中时，更加紧缩。

**单路和双路的选择**

-  max_length_for_sort_data 比查询字段的总长度大，使用单路排序模式；
-  max_length_for_sort_data 比查询字段的总长度小，使用回表排序模式。



如果 MySQL 排序内存有条件可以配置比较大，可以适当增大 max_length_for_sort_data 的值，让优化器优先选择全字段排序，把需要的字段放到 sort_buffer 中，这样排序后就会直接从内存里返回查询结果了。

## order by 优化

###  添加合适索引

1、 排序字段添加索引

2、多个字段排序添加联合索引

3、先等值查询再排序，在条件字段和排序字段添加联合索引

### 去掉不必要的返回字段

过多返回字段可能需要扫描索引再回表，成本全表扫描更高。

```mysql
select id,a,b from t1 order by a,b; /* 根据a和b字段排序查出id,a,b字段的值 */
```

### 修改参数

max_length_for_sort_data：可以适当加大 max_length_for_sort_data 的值

sort_buffer_size：适当加大 sort_buffer_size 的值，尽可能让排序在内存中完成。

### 无法使用索引

1、 使用范围查询再排序：

```mysql
select id,a,b from t1 where a>9000 order by b;
```

2、ASC 和 DESC 混合使用

```mysql
select id,a,b from t1 order by a asc,b desc;
```

## group by优化

默认情况，会对 group by 字段排序，因此优化方式与 order by 基本一致。

如果目的只是分组而不用排序，可以指定`order by null`禁止排序。

