---
title: 多阶段构建
top: false
cover: false
toc: true
mathjax: true
date: 2021-06-07 22:03:15
password:
summary:
tags:
- docker
categories:
- docker
---

## 引入

　在构建镜像过程中，我们可能只需要某些镜像的产物，比如在运行一个go程序需要先go程序包编译后才运行，如果在一个镜像里面完成，先要经过安装编译环境，程序编译完再安装运行环境，最后运行程序，这样的镜像体积往往比较大，不利于我们使用。而真正我们需要的镜像是只有程序包和运行环境，编译环境的构建在运行容器时候是不需要的，所以Docker提供了一种解决方案就是multi-stage（多阶段构建）。

## 实践

Docker允许多个镜像的构建可以使用同一个Dockerfile，每个镜像构建过程可以称之为一个stage，简单理解就是一个FROM指令到下一个FROM指令，而每个stage可使用上一个stage过程的产物或环境（其实还支持其他镜像的），这样一来，最终所得镜像体积相对较小。

不仅如此多阶段构建同样可以很方便地将多个彼此依赖的项目通过一个Dockerfile就可轻松构建出期望的容器镜像，而不用担心镜像太大、项目环境依赖等问题。

通过上述介绍，我们可以在第一个stage将go程序编译得到编译后程序包，然后在第二个stage中直接拷贝编译好的go程序包到运行环境中，最后的镜像中就只有程序包和运行环境。以下作为示例：

```
FROM golang:1.7.3
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
# ！！！
COPY --from=0 /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]
```

在以上Dockerfile中存在两个FROM指令，也就是两个stage，第一个stage用于构建产物，而在第二个stage中使用COPY --from=0 意思将第一个stage中的/go/src/github.com/alexellis/href-counter/app拷贝到.目录，第二个stage仅仅相当于执行copy就有了构建产物，不用在安装编译环境，镜像会很缩小。 

### 命名stage

默认情况下，stage未命名，可以通过整数来引用它们，第一个stage表示0，第二个表1以此类推。 但是，当有多个stage时候，这样会显得麻烦，Docker提供AS 语法可以为stage命名：

```
FROM golang:1.7.3 as builder
```

然后在另一个stage中使用：

```
COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
```

## 构建镜像建议

1. 基础镜像尽量选择比体积较小的镜像，如每个官方发行的alpine镜像。虽然这版本镜像比较小，但是与之带来的是利用该类镜像运行的容器中排错的命令很少;
2. 使用RUN指令时候，尽量把多个RUN指令合并为一个，通常做法是使用&&符号;
3. 通过multi-stage方法减少一些不必要使用的环境来减小镜像;
4. 安装完成软件同时删除一些不需要的文件或目录； 

## 参考

https://www.cnblogs.com/wdliu/p/10469257.html