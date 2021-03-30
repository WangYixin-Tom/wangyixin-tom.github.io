---
title: kafka授权
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-29 21:47:25
password:
summary:
tags:
- kafka
categories:
- kafka
---

授权，一般是指对与信息安全或计算机安全相关的资源授予访问权限，特别是存取控制。

Kafka用的是 ACL 模型，规定了什么用户对什么资源有什么样的访问权限。

## kafka-acls 脚本

```shell
# Alice 增加了集群级别的所有权限
$ kafka-acls --authorizer-properties zookeeper.connect=localhost:2181 --add --allow-principal User:Alice --operation All --topic '*' --cluster


$ kafka-acls --authorizer-properties zookeeper.connect=localhost:2181 --add --allow-principal User:'*' --allow-host '*' --deny-principal User:BadUser --deny-host 10.205.96.119 --operation Read --topic test-topic
```

