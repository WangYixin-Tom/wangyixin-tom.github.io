---
title: kafka认证
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-29 21:37:09
password:
summary:
tags:
- kafka
categories:
- kafka
---

认证要解决的是你要证明你是谁的问题，授权要解决的则是你能做什么的问题。

## Kafka 认证机制

**基于 SSL 的认证**主要是指 Broker 和客户端的双路认证。

客户端认证 Broker 的证书，且Broker 也要认证客户端的证书。

**基于 SASL 的安全认证机制**

SASL 是提供认证和数据安全服务的框架。Kafka 支持的 SASL 机制有 5 种：

GSSAPI：也就是 Kerberos 使用的安全接口，是在 0.9 版本中被引入的。

PLAIN：是使用简单的用户名 / 密码认证的机制，在 0.10 版本中被引入。

SCRAM：主要用于解决 PLAIN 机制安全问题的新机制，是在 0.10.2 版本中被引入的。

OAUTHBEARER：是基于 OAuth 2 认证框架的新机制，在 2.0 版本中被引进。

Delegation Token：补充现有 SASL 机制的轻量级认证机制，是在 1.1.0 版本被引入的。

## 认证机制的比较

可以使用 SSL 来做通信加密，使用 SASL 来做 Kafka 的认证实现。

[](compare.jpg)

