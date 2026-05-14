---
title: "gRPC"
description: "gRPC"
date: 2022-06-08 18:10:00
slug: gRPC-intro
image:
categories:
    - Go
tags: ["Go"]

---



### 安装安装Protocol Buffers v3

#### 安装 protobuf 的编译器 protoc



根据电脑环境选择对应的下载

下载地址：https://github.com/protocolbuffers/protobuf/releases

protocol-buffers 官方Go教程：

https://developers.google.com/protocol-buffers/docs/gotutorial

下载后解压：

其中：

- bin 目录下的 protoc 是可执行文件。
- include 目录下的是 google 定义的`.proto`文件，我们`import "google/protobuf/timestamp.proto"`就是从此处导入。

之后将bin下的`protoc`添加到环境变量中。

```sh
# 运行打印版本号，查看是否安装成功
protoc --version
```

##### 安装插件

protoc 需要 protoc-gen-go 来完成 Go 语言的代码转换，因此我们需要安装 protoc 和 protoc-gen-go 。

注意：

`github.com/golang/protobuf/protoc-gen-go`和`google.golang.org/protobuf/cmd/protoc-gen-go`是不同的。区别在于前者是旧版本，后者是google接管后的新版本，他们之间的API是不同的，也就是说用于生成的命令，以及生成的文件都是不一样的。因为目前的gRPC-go源码中的example用的是后者的生成方式，所以这里提供后者安装方式。

- 安装go语言插件：

```sh
go install google.golang.org/protobuf/cmd/protoc-gen-go
```

该插件会根据`.proto`文件生成一个后缀为`.pb.go`的文件，包含所有`.proto`文件中定义的类型及其序列化方法。

- 安装grpc插件：

  ```sh
  go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
  ```

该插件会生成一个后缀为`_grpc.pb.go`的文件，其中包含：

- 一种接口类型(或存根) ，供客户端调用的服务方法。
- 服务器要实现的接口类型。

上述命令会默认将插件安装到`$GOPATH/bin`，为了`protoc`编译器能找到这些插件，请确保你的`$GOPATH/bin`在环境变量中。

```sh
# 检查是否安装成功
# protoc-gen-go安装的版本
protoc-gen-go --version

# protoc-gen-go-grpc 安装的版本
protoc-gen-go-grpc --version
```

### gRPC示例



目录结构：

```sh
├─client
	-- main.go
├─helloworld
	-- helloworld.proto
	-- helloworld.pb.go
	-- helloworld_grpc.pb.go
└─server
	-- main.go

```

具体步骤分为四步：

- 定义 gRPC 服务
- 生成客户端和服务器代码
- 实现 gRPC 服务
- 实现 gRPC 客户端

新建一个`grpc-demo`项目，进入该目录使用`go mod init grpc-demo`初始化项目

#### 定义 gRPC 服务

创建 helloworld 目录，新建文件 helloworld.proto。

`grpc-demo/helloworld/helloworld.proto`文件内容如下：

```protobuf

syntax = "proto3"; // // 版本声明，使用Protocol Buffers v3版本

/*
go_package 是必需的设置，而且 go_package 的值必须是包导入的路径
;逗号前面的代码表示其他proto引用这个proto时使用的go包路径
;逗号后面的代码这个proto生成go代码时package的名称

*/
option go_package = "grpc-demo/helloworld;helloworld";
// package 关键字指定生成的.pb.go 文件所在的包名
package helloworld;


// gRPC 支持定义 4 种类型的服务方法，分别是简单模式、服务端数据流模式、客户端数据流模式和双向数据流模式。
// 本例使用简单模式

// 定义服务
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
// 请求消息
message HelloRequest {
  string name = 1;
}
// 响应消息
// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```





#### 生成服务器和客户端代码

在根目录`grpc-demo`下执行命令：

```sh
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative helloworld/helloworld.proto
```

生成如下2个文件：

- helloworld.pb.go
- helloworld_grpc.pb.go

#### 实现服务端

创建 `server` 目录，新建文件 main.go。

`grpc-demo/server/main.go`文件内容如下：

```go
// Package main implements a server for Greeter service.
package main

import (
	"context"
	"log"
	"net"

	pb "grpc-demo/helloworld"

	"google.golang.org/grpc"
)

const (
	port = ":50051"
)

// server is used to implement helloworld.GreeterServer.
type server struct {
	pb.UnimplementedGreeterServer
}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	log.Printf("Received: %v", in.GetName())
	return &pb.HelloReply{Message: "Hello " + in.GetName()}, nil
}

func main() {
	//监听本地端口
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	// 创建gRPC服务器， 创建一个 gRPC Server 实例
	s := grpc.NewServer()
	// 将该服务注册到 gRPC 框架中
	pb.RegisterGreeterServer(s, &server{})
	// 启动服务
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}

```

在`grpc-demo/server/`目录下执行`go run main.go`启动gRPC服务。

#### 实现客户端

创建 `client` 目录，新建文件 main.go。

`grpc-demo/client/main.go`文件内容如下：

```go
// Package main implements a client for Greeter service.
package main

import (
	"context"
	"log"
	"os"
	"time"

	pb "grpc-demo/helloworld"

	"google.golang.org/grpc"
)

const (
	address     = "localhost:50051"
	defaultName = "world"
)

func main() {
	// Set up a connection to the server.
	// 创建一个 gRPC 连接，跟服务端进行通信
	// 创建连接时可以指定不同的选项
	conn, err := grpc.Dial(address, grpc.WithInsecure(), grpc.WithBlock())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := pb.NewGreeterClient(conn)

	// Contact the server and print out its response.
	name := defaultName
	if len(os.Args) > 1 {
		name = os.Args[1]
	}
	// 创建连接后就可以像调用本地方法一样调用远程方法
	// 执行RPC调用并打印收到的响应数据
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()
	r, err := c.SayHello(ctx, &pb.HelloRequest{Name: name})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Printf("Greeting: %s", r.Message)
}

```

在`grpc-demo/client/`目录下执行`go run main.go`发起gRPC调用。



