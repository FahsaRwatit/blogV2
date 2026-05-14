---
title: "Windows中使用go-wrk进行http服务压力测试"
description: "Windows中使用go-wrk进行http服务压力测试"
date: 2022-05-10 18:10:00
slug: windows-go-wrk
image:
categories:
    - Go
tags: ["Go", "go-micro"]

---



### 环境准备

```sh
# window 安装Protobuf，下载地址：https://github.com/protocolbuffers/protobuf/releases

# 安装依赖
go get -u google.golang.org/protobuf/proto
go get -u google.golang.org/protobuf/proto

//注意：不要安装下面这个，下面是Micro3.0 云平台的安装工具
//go get github.com/micro/micro/v3/cmd/protoc-gen-micro

//安装asim/go-micro   这个是go micro 3.0 框架，
go get github.com/asim/go-micro/cmd/protoc-gen-micro/v3

# 安装micro v3
# 安装protoc-gen-micro,  GOPATH/bin目录下会有protoc-gen-micro.exe
go install github.com/micro/micro/v3/cmd/protoc-gen-micro@master

# 安装micro v3,  GOPATH/bin目录下会有micro.exe
go install github.com/micro/micro/v3@latest


-----------------------
# 如果一不小心安装了Micro3.0，可以使用 go clean -i 移除
go get github.com/micro/micro/v3/cmd/protoc-gen-micro
go clean -i github.com/micro/micro/v3/cmd/protoc-gen-micro

# 安装protoc-gen-gofast插件
go install github.com/gogo/protobuf/protoc-gen-gofast@latest

```

安装完毕后检查是否包含以下程序：

```sh
GOPATH/bin 目录下
micro.exe
protoc.exe
protoc-gen-go.exe
protoc-gen-go-grpc.exe
protoc-gen-micro.exe
protoc-gen-gofast.exe
```





### protoc生成go文件
```shell
protoc --proto_path=. --micro_out=. --gofast_out=:. proto/helloworld.proto
```







参考：https://www.jianshu.com/p/99cf70a5fefe










