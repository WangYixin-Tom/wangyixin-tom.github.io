---
title: redis网络IO模型
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-26 22:06:23
password:
summary:
tags:
- redis
categories:
- redis
---

## 单线程

Redis 是单线程，主要是指 Redis 的网络 IO 和键值对读写是由一个线程来完成的。持久化、异步删除、集群数据同步等，其实是由额外的线程执行的。

避免了多线程编程模式面临的共享资源的并发访问控制问题。

## 多路复用机制

一个线程处理多个 IO 流（select/epoll）：在 Redis 只运行单线程的情况下，该机制允许内核中，同时存在多个监听套接字和已连接套接字。内核会一直监听这些套接字上的连接请求或数据请求。一旦有请求到达，就会交给 Redis 线程处理，这就实现了一个 Redis 线程处理多个 IO 流的效果。

为了在请求到达时能通知到 Redis 线程，select/epoll 提供了基于事件的回调机制，即针对不同事件的发生，调用相应的处理函数。