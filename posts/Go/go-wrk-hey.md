---
title: "使用wrk和hey进行压测示例"
description: "使用wrk和hey进行压测示例"
date: 2022-04-10 18:10:00
slug: go-wrk-hey
image:
categories:
    - Go
tags: ["Go"]


---

## 使用wrk进行压测

### 安装wrk

```shell
$ sudo apt-get install build-essential libssl-dev git -y
$ git clone https://github.com/wg/wrk.git wrk
$ cd wrk
$ make
$ sudo cp wrk /usr/local/bin
```

### 编写测试脚本

```lua
# test.lua
wrk.method = "POST"
wrk.headers["Content-Type"] = "application/json"
wrk.body = '{"id":1,"name":"test"}'
wrk.path = "/api/user"
```

### 运行

`$ wrk -c 100 -t 10 -d 30s -s test.lua http://localhost:8888`

解释：
-c: 连接数，即并发数
-t: 线程数
-d: 压测时间
-s: 指定测试脚本
http://localhost:8888: 待测试的接口地址

## 使用hey进行压测

### 安装hey

```shell
$ go get -u github.com/rakyll/hey
```

### 运行hey进行压测

```shell
$ hey -c 100 -n 10000 -m POST -H "Content-Type: application/json" -D '{"id":1,"name":"test"}' http://localhost:8888/api/user
```

解释：
-c: 连接数，即并发数
-n: 请求总数
-m: 请求方法
-H: 请求头
-D: 请求体
http://localhost:8888/api/user: 待测试的接口地址
以上示例中，我们分别使用了wrk和hey进行了压测。这些工具在go-zero中使用非常方便，可以快速测试我们的API性能并发现问题。
