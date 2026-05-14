---
title: "Windows中使用go-wrk进行http服务压力测试"
description: "Windows中使用go-wrk进行http服务压力测试"
date: 2022-05-10 18:10:00
slug: windows-go-wrk
image:
categories:
    - Go
tags: ["Go"]


---

go-wrk是Go语言版本的wrk，Github地址：https://github.com/adjust/go-wrk

可在Windows中进行压力测试

**安装**

```go
go get github.com/adeven/go-wrk
```

进入 `$GOPATH/bin `

用法：go-wrk [flags] url

运行：

```go
go-wrk -t=8 -c=100 -n=10000 "http://localhost:8081/api/v1/refresh_token"
```

结果显示：

```go
==========================BENCHMARK==========================
URL:                            http://localhost:8081/api/v1/community/1

Used Connections:               100
Used Threads:                   8
Total number of calls:          1000

===========================TIMINGS===========================
Total time passed:              0.45s
Avg time per request:           38.84ms
Requests per second:            2245.48
Median time per request:        37.56ms
99th percentile time:           66.12ms
Slowest time for request:       75.00ms

=============================DATA=============================
Total response body sizes:              63000
Avg response body per request:          63.00 Byte
Transfer rate per second:               141465.27 Byte/s (0.14 MByte/s)
==========================RESPONSES==========================
20X Responses:          1000    (100.00%)
30X Responses:          0       (0.00%)
40X Responses:          0       (0.00%)
50X Responses:          0       (0.00%)
Errors:                 0       (0.00%)
```



#### 常见参数

```bash
-H="User-Agent: go-wrk 0.1 bechmark\nContent-Type: text/html;": 由'\n'分隔的请求头
-c=100: 使用的最大连接数
-k=true: 是否禁用keep-alives
-i=false: if TLS security checks are disabled
-m="GET": HTTP请求方法
-n=1000: 请求总数
-t=1: 使用的线程数
-b="" HTTP请求体
-s="" 如果指定，它将计算响应中包含搜索到的字符串s的频率
```

#### 显示结果

```bash
Used Connections:               使用连接数 
Used Threads:                   使用线程数
Total number of calls:          请求总数

Total time passed: 总通过时间
Avg time per request:每个请求的平均时间
Requests per second: 每秒请求数
Median time per request:每个请求的平均时间
99th percentile time: 第99百分位时间
Slowest time for request: 请求的最慢时间

Total response body sizes: 总响应体尺寸
Avg response body per request: 每个请求的平均响应正文
Transfer rate per second: 每秒传输速率
```