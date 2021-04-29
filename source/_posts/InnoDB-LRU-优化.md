---
title: InnoDB LRU 优化
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-21 20:42:36
password:
summary:
tags:
- mysql
categories:
- mysql
---

InnoDB内存管理用的是最近最少使用 （Least Recently Used）算法，这个算法的核心就是淘汰最久未使用的数据。为了应对全表扫描的影响，InnoDB对LRU算法做了改进。

在InnoDB实现上，按照5:3的比例把整个LRU链表分成了young区域和old区域。图中LRU_old指向的就是old区域的第一个位置，是整个链表的5/8处。也就是说，靠近链表头部的5/8是young区域，靠近链表尾部的3/8是old区域。

1、访问young区域，因此和优化前的LRU算法一样，将其移到链表头部

2、要访问一个新的不存在于当前链表的数据页，这时候依然是淘汰掉数据页Pm，但是新插入的数据页Px，是放在LRU_old处。

3、处于old区域的数据页，每次被访问的时候都要做下面这个判断：

- 若这个数据页在LRU链表中存在的时间超过了1秒，就把它移动到链表头部；
- 如果这个数据页在LRU链表中存在的时间短于1秒，位置保持不变。1秒这个时间，是由参数innodb_old_blocks_time控制的。