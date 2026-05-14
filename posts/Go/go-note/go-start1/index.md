---
title: Go开始
description: Go
date: 2019-03-12 15:10:00
slug: go-start1
image:
categories:
    - Go
tags: ["Go"]

---



### 程序入口

1、必须是 main 包：package main

2、必须是 main 方法：func main（）

3、文件名不一定是 main.go

### 退出返回值

**与其他语言的差异**

- Go 中main 函数不支持任何返回值
- 通过os.Exit来返回状态
- main 函数不支持传入参数
- 在程序中直接通过os.Args获取命令行参数

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	fmt.Println(os.Args)
	fmt.Println(len(os.Args))
	if len(os.Args) > 1 {
		fmt.Println("hello world:" + os.Args[1])
	}
	os.Exit(0)
}

```

### 编写测试程序

1、源码文件以_test结尾：xxx_test.go

2、测试方法名以Test开头：func Testxxx(t *testing.T){...}

### 变量赋值

**与其他语言的差异**

- 赋值可以进行自动类型推断
- 在一个赋值语句中可以对多个变量同事赋值









