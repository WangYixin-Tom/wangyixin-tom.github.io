---
title: kafka Broker请求处理
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-28 17:58:17
password:
summary:
tags:
- kafka
categories:
- kafka
---

所有的请求都是通过 TCP 网络以 Socket 的方式进行通讯的。

Kafka 使用的是 Reactor 模式处理请求。

**Reactor 模式**是事件驱动架构的一种实现方式，特别适合应用于处理多个客户端并发向服务器端发送请求的场景。多个客户端会发送请求给到 Reactor。Reactor 有个请求分发线程 Dispatcher，它会将不同的请求下发到多个工作线程中处理。工作线程可以根据实际业务处理需要任意增减，从而动态调节系统负载能力。

![](reactor.jpg)

Kafka 提供了 Broker 端参数 `num.network.threads`，用于调整该**网络线程池**的线程数。其默认值是 3，表示每台 Broker 启动时会创建 3 **个网络线程**，专门处理客户端发送的请求。

![](work.jpg)

当网络线程拿到请求后，它不是自己处理，而是将请求放入到一个**共享请求队列**中。Broker 端还有个 **IO 线程池**，负责从该队列中取出请求，执行真正的处理--如果是 PRODUCE 生产请求，则将消息写入到底层的磁盘日志中；如果是 FETCH 请求，则从磁盘或页缓存中读取消息。

Broker 端参数 `num.io.threads` 控制了IO线程池中的线程数。默认值是 8，表示每台 Broker 启动后自动创建 8 个 IO 线程处理请求。

**Purgatory**

缓存延时请求（Delayed Request）,针对一时未满足条件不能立刻处理的请求。

比如设置了 acks=all 的 PRODUCE 请求，一旦设置了 `acks=all`，那么该请求就必须等待 ISR 中所有副本都接收了消息后才能返回，此时处理该请求的 IO 线程就必须等待其他 Broker 的写入结果。当请求不能立刻处理时，它就会暂存在 Purgatory 中。一旦完成，IO 线程会继续处理该请求，并将 Response 放入对应网络线程的响应队列中。