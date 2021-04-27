---
title: redis变慢以及优化方法
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-26 21:42:19
password:
summary:
tags:
- redis
categories:
- redis
---

## 确定问题

1、查看 Redis 的响应延迟。
2、基于当前环境下的 Redis 基线性能做判断
基线性能是系统在低压力、无干扰下的基本性能，Redis 运行时延迟是其基线性能的 2 倍及以上，可认定 Redis 变慢了。


## 问题定位

1、通过 Redis 日志，或者是 latency monitor 工具，查询变慢的请求，确认是否采用了复杂度高的慢查询命令。
2、检查业务代码在使用 EXPIREAT 命令设置 key 过期时间时，是否使用了相同的 UNIX 时间戳，有没有使用 EXPIRE 命令给批量的 key 设置相同的过期秒数。从而造成大量 key 在同一时间过期，导致性能变慢。删除操作是阻塞的（Redis 4.0 后可以用异步线程机制来减少阻塞影响）
3、检查是否使用了慢查询命令，KEYS *xxx


## 优化
1.a.用其他高效命令代替。比如说，如果你需要返回一个 SET 中的所有成员时，不要使用 SMEMBERS 命令，而是要使用 SSCAN 多次迭代返回，避免一次返回大量数据，造成线程阻塞。
1.b.当你需要执行排序、交集、并集操作时，可以在客户端完成，而不要用 SORT、SUNION、SINTER 这些命令，以免拖慢 Redis 实例。

2.如果一批 key 的确是同时过期，你还可以在 EXPIREAT 和 EXPIRE 的过期时间参数上，加上一个一定大小范围内的随机数

3.获取整个实例的所有key，建议使用SCAN命令代替。客户端通过执行`SCAN $cursor COUNT $count`可以得到一批key以及下一个游标`$cursor`，然后把这个`$cursor`当作SCAN的参数，再次执行，以此往复，直到返回的`$cursor`为0时，就把整个实例中的所有key遍历出来了。