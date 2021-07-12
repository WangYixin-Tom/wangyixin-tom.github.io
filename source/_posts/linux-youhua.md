---
title: linux优化方法
top: false
cover: false
toc: true
mathjax: true
date: 2021-07-08 07:44:42
password:
summary:
tags:
- linux
categories:
- linux
---

## CPU

### 高cpu占用率的进程和线程

```shell
top 				# 查看高CPU进程
top -H -p pid		# 查看高CPU线程
```

### CPU占用资源低但是系统响应速度很慢可能是什么问题

等待磁盘I/O完成的进程过多，导致进程队列长度过大，但是cpu运行的进程却很少，这样就体现到负载过大了，cpu使用率低。

负载就是cpu在一段时间内正在处理以及等待cpu处理的进程数之和的统计信息，也就是cpu使用队列的长度统计信息，这个数字越小越好



