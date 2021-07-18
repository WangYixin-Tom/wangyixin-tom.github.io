---
title: docker必知必会
top: false
cover: false
toc: true
mathjax: false
date: 2021-07-07 15:19:32
password:
summary: docker重要知识点
tags:
- interview
categories:
- interview
---



## 基本概念

### 什么是容器

容器是一种轻量级、可移植、自包含的软件打包技术，使应用程序可以在几乎任何地方以相同的方式运行。

## 必要性

**容器使软件具备了超强的可移植能力**。

Docker 解决了环境配置问题，可以简化应用部署。它是一种虚拟化技术，对进程进行隔离，被隔离的进程独立于宿主操作系统和其它隔离的进程

### **vs虚拟机**

虚拟机也是一种虚拟化技术，通过模拟硬件，并在硬件上安装操作系统来实现。容器在 Host 操作系统的用户空间中运行，与操作系统的其他进程隔离。

![](docker_vm.png)

#### 启动速度

虚拟机需要先启动虚拟机的操作系统，再启动应用，速度慢；

Docker 相当于启动宿主操作系统上的一个进程，不需要启动整个操作系统。

#### 占用资源

虚拟机是一个完整的操作系统，需要占用大量的磁盘、内存和 CPU 资源，一台机器只能开启几十个的虚拟机。

Docker 只是一个进程，只需要将应用以及相关的组件打包，在运行时占用很少的资源，一台机器可以开启成千上万个 Docker。

### 优点

除了启动速度快以及占用资源少之外，Docker 具有以下优势：

**更容易迁移**

提供一致性的运行环境。已经打包好的应用可以在不同的机器上进行迁移，而不用担心环境变化导致无法运行。

**更容易维护**

使用分层技术和镜像，使得应用可以更容易复用重复的部分。复用程度越高，维护工作也越容易。

**更容易扩展**

可以使用基础镜像进一步扩展得到新的镜像，并且官方和开源社区提供了大量的镜像，通过扩展这些镜像可以非常容易得到我们想要的镜像。

## 使用场景

**持续集成**

持续集成指的是频繁地将代码集成到主干上，这样能够更快地发现错误。

Docker 具有轻量级以及隔离性的特点，在将代码集成到一个 Docker 中不会对其它 Docker 产生影响。

**提供可伸缩的云服务**

根据应用的负载情况，可以很容易地增加或者减少 Docker。

**搭建微服务架构**

Docker 轻量级的特点使得它很适合用于部署、维护、组合微服务。

## 基本命令

### 常用命令

```shell
FROM
指定 base 镜像。

MAINTAINER
设置镜像的作者，可以是任意字符串。

COPY
将文件从build context复制到镜像。

ADD
与 COPY 类似，从 build context 复制文件到镜像。如果 src 是归档文件（tar, zip等），文件会被自动解压到 dest。

ENV
设置环境变量，环境变量可被后面的指令使用。

EXPOSE
指定容器中的进程会监听某个端口，Docker可以将该端口暴露出来。

VOLUME
将文件或目录声明为 volume

WORKDIR
为后面的 RUN, CMD, ENTRYPOINT, ADD 或 COPY 指令设置镜像中的当前工作目录。

RUN
在容器中运行指定的命令。

CMD
容器启动时运行指定的命令。
Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效。CMD 可以被 docker run 之后的参数替换。

ENTRYPOINT
设置容器启动时运行的命令。
Dockerfile 中可以有多个 ENTRYPOINT 指令，但只有最后一个生效。CMD 或 docker run 之后的参数会被当做参数传递给 ENTRYPOINT。
```

### RUN、CMD和ENTRYPOINT区别

1. RUN 执行命令并创建新的镜像层，常用于安装软件包。
2. CMD 设置容器启动后默认执行的命令及其参数，此命令会在容器启动且 docker run 没有指定其他命令时运行。如果 docker run 指定了其他命令，CMD 指定的默认命令将被忽略。如果 Dockerfile 中有多个 CMD 指令，只有最后一个 CMD 有效。
3. ENTRYPOINT 配置容器启动时运行的命令。 ENTRYPOINT 指令可让容器以应用程序或者服务的形式运行。ENTRYPOINT 不会被忽略，一定会被执行，即使运行 docker run 时指定了其他命令。

### 解释一下dockerfile的ONBUILD指令？

当镜像用作另一个镜像构建的基础时，向镜像添加将在稍后执行的触发指令。

如果要构建将用作构建其他镜像的基础的镜像，这将非常有用。

### 构建docker镜像应该遵循哪些原则

- 尽量选取满足需求但较小的基础系统镜像。
- 使用RUN指令时候，尽量把多个RUN指令合并为一个，通常做法是使用&&符号；COPY、ADD和RUN语句会向镜像中添加新层
- 清理编译生成文件、安装包的缓存等临时文件。
- 通过multi-stage方法减少一些不必要使用的环境来减小镜像;
- 安装各个软件时候要指定准确的版本号，并避免引入不需要的依赖。
- 从安全的角度考虑，应用尽量使用系统的库和依赖。
- 使用dockerfile创建镜像时候要添加.dockerignore文件或使用干净的工作目录。

### 如何控制容器占用系统资源（CPU，内存）的份额

docker create命令创建容器或使用docker run 创建并运行容器的时候，可以使用-c|-cpu-shares[=0]参数来调整同期使用CPU的权重，使用-m|-memory参数来调整容器使用内存的大小。

### 如何停止所有正在运行的容器？

使用`docker kill $(sudo docker ps -q)`

### 如何清理批量后台停止的容器？

使用`docker rm $（sudo docker ps -a -q）`

### 批量删除所有已经退出的容器

`docker rm -v $(docker ps -aq -f status=exited)`

## 镜像

- 可将 Docker 镜像看着只读模板，通过它可以创建 Docker 容器。
- Docker 容器就是 Docker 镜像的运行实例。

镜像包含着容器运行时所需要的代码以及其它组件，它是一种分层结构，每一层都是只读的（read-only layers）。构建镜像时，会一层一层构建，前一层是后一层的基础。镜像的这种分层存储结构很适合镜像的复用以及定制。

构建容器时，通过在镜像的基础上添加一个可写层（writable layer），用来保存着容器运行过程中的修改。

### **base镜像**

用户空间的文件系统是 rootfs，包含我们熟悉的 /dev，/proc， /bin 等目录。

对于 base 镜像来说，底层直接用 Host 的 kernel，自己只需要提供 rootfs 就行了。

**base 镜像提供的是最小安装的 Linux 发行版**。不同 Linux 发行版的区别主要就是 rootfs。

Docker 可以同时支持多种 Linux 镜像，模拟出多种操作系统环境。

### 分层

新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层。最大的一个好处就是共享资源

所有对容器的改动，无论添加、删除、还是修改文件，都只会发生在容器层中。

只有当需要修改时才复制一份数据，这种特性被称作 Copy-on-Write。可见，容器层保存的是镜像变化的部分，不会对镜像本身进行任何修改。

### 镜像的缓存特性

Docker 会缓存已有镜像的镜像层，构建新镜像时，如果某镜像层已经存在，就直接使用，无需重新创建。

### 多阶段构建

 `Dockerfile` 中使用多个 FROM 语句。每个 FROM 指令都可以使用不同的基础镜像，并表示开始一个新的构建阶段。你可以很方便的将一个阶段的文件复制到另外一个阶段，在最终的镜像中保留下你需要的内容即可。

```
COPY --from=build-env /go/src/app/app-server /usr/local/bin/app-server
```

## 容器

### attach 与 exec 主要区别

1. attach 直接进入容器 **启动命令**的终端，不会启动新的进程。
2. exec 则是在容器中打开新的终端，并且可以启动新的进程。

> 在终端中查看启动命令的输出，用 attach；其他情况使用 exec。
>
> 只是为了查看启动命令的输出，可以使用 `docker logs` 命令

### 底层技术

- cgroup （Control Group）实现资源限额，通过 cgroup 可以设置进程使用 CPU、内存 和 IO 资源的限额。 
- namespace 实现资源隔离。

**六种 namespace**

- Mount namespace 让容器看上去拥有整个文件系统。
- UTS namespace 让容器有自己的 hostname。
- IPC namespace 让容器拥有自己的共享内存和信号量来实现进程间通信
- PID namespace：容器拥有自己独立的一套 PID
- Network namespace 让容器拥有自己独立的网卡、IP、路由等资源。
- User namespace 让容器能够管理自己的用户

## 网络

### 原生网络/单个 host 上的容器网络

none 网络：什么都没有的网络。挂在这个网络下的容器除了 lo，没有其他任何网卡。

host 网络：容器共享 Docker host 的网络栈，容器的网络配置与 host 完全一样。性能好，牺牲一些灵活性要考虑端口冲突问题。

bridge 网络：veth pair，一段挂在网桥 `docker0` 上，通信需要使用同一网络下的网卡

### user-defined 网络驱动

bridge：需要创建网桥。创建容器，可基于网桥所在网络指定IP。跨容器通信需要使用同一网桥下的网络。

overlay ：

macvlan：

### 容器间通信

- IP：使用同一网桥下的网络
- DNS：docker daemon 实现了一个内嵌的 DNS server，使容器可以直接通过“容器名”通信。**只能在 user-defined 网络中使用**。默认的 bridge 网络是无法使用 DNS 的。
- joined容器：两个或多个容器共享一个网络栈，共享网卡和配置信息，joined 容器之间可以通过 127.0.0.1 直接通信，适合web server 与 app server容器。

### 容器访问外部世界

**容器默认就能访问外网**。如果网桥 `docker0` 收到来自 172.17.0.0/16 网段的外出包，把它交给 MASQUERADE 处理。而 MASQUERADE 的处理方式是将包的源地址替换成 host 的地址发送出去，**即做了一次网络地址转换（NAT）**

### 外部世界如何访问容器

docker 可将容器对外提供服务的端口映射到 host 的某个端口，外网通过该端口访问容器。容器启动时通过`-p`参数映射端口。docker-proxy 监听，某一个端口并转发给容器。

## 存储

### storage driver

管理的镜像层和容器层，对于某些容器，直接将数据放在由 storage driver 维护的层中是很好的选择，比如那些无状态的应用。无状态意味着容器没有需要持久化的数据，随时可以从镜像直接创建。

### Data Volume

Docker Host 文件系统中的目录或文件，能够直接被 mount 到容器的文件系统中。Data Volume 是目录或文件，而非没有格式化的磁盘（块设备）。容器可以读写 volume 中的数据。volume 数据可以被永久的保存，即使使用它的容器已经销毁。

**bind mount**
将 host 上已存在的目录或文件 mount 到容器，可设置为只读，默认为读写权限
**用途**：源代码/数据备份，删除容器后还会保留，支持单个文件
**缺点**：移植性弱，与 host path 绑定

**docker managed volume**
原有数据复制到 volume，移植性强，无需指定 host 目录，不支持单个文件，无控制权限

### 数据共享

**容器与 host 共享数据**

- bind mount

- docker managed volume

- docker cp 


**容器之间共享数据**

- bind mount多个容器
- volume container（专门为其他容器提供 volume 的容器）：容器与 host 的解耦
- data-packed volume container：将数据打包到镜像中，然后通过 docker managed volume 共享，不依赖 host 提供数据，具有很强的移植性，非常适合只使用静态数据的场景