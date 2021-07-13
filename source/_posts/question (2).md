# datastructrue
跳表怎么实现
虚拟内存是什么，逻辑地址是怎么映射到物理地址的

# changjing

超买超卖问题

路由器和交换机具体都实现了什么功能？路由选择是如何实现的？

用过抖音，微信吧，当你的手机打开抖音和微信时，你觉得后台在做一些什么事情？

# kuozhan

线程池是什么，讲讲里面具体的参数，线程池分什么种类，如何自己实现一个线程池

地址映射有哪些算法


用cookie和session实现用户登录的过程是咋样的，

说一下AMQP协议是怎么定义一个MQ的
ThreadLocal有了解吗？说一下注意事项？
说说微服务中的降级，熔断与限流

token的设计应该考虑哪些因素？

token怎么获取的，怎么生成的，怎么防止被盗用（这里几乎每一家面试都会问，使用httponly解决）

jWT的原理，为什么可以被用于验证，和session方式的区别
描述一下session的产生，和与前台的交互方式，sessionid。从整体的角度描述一下session或者jwt方式的登陆
分布式session，各种解决方案的优点和缺点
如果使用JWT的方式，如何存储大量的数据（如购物车）
JWT的结构，JWT存储信息的长度收什么限制（HTTP头的长

paxos和zab的区别
zoo[keep](https://www.nowcoder.com/jump/super-jump/word?word=keep)er的zab算法

paxos的理解


paxos：多个proposer发请提议（每个提议有id+value），acceptor接受最新id的提议并把之前保留的提议返回。当超过半数的accetor返回某个提议时，此时要求value修改为propeser历史上最大值，propeser认为可以接受该提议，于是广播给每个acceptor，acceptor发现该提议和自己保存的一致，于是接受该提议并且learner同步该提议。

raft：raft要求每个节点有一个选主的时间间隔，每过一个时间间隔向master发送心跳包，当心跳失败，该节点重新发起选主，当过半节点响应时则该节点当选主机，广播状态，然后以后继续下一轮选主。

讲解Raft算法 有三个角色，follower，candidate，leader。Raft最多能容忍(n-1)/2 的错误，假设系统有三个节点，A B C 一开始都是follower的状态，每一个节点都会倒计时，当倒计时完，就给各个节点发送 信号，争取当 leader。假设A节点最快完成，成为了leader，剩余两个节点就会变成Follower状态，从leader节点读取log文件。

ip分片与重组，什么时候分片？在哪里重组？依据什么重组？怎么判断重组全部完成？
MSS的大小是控制什么的？

1.5selec需要把文件描述符写入内存吗？

NIO 的核心理念是什么？
NIO 多路复用select() epoll() 演进流程 同步到异步

CAS和ABA问题



Python中static_method 、 class_mathod 、和普通method有什么区别

Reactor与proactor的区别？

