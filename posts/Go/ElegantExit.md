---
title: "Golang优雅退出程序"
description: "Golang优雅退出程序"
date: 2022-12-08 18:10:00
slug: ElegantExit
image:
categories:
    - Go
tags: ["Go"]


---

**golang中对信号的处理主要使用os/signal包中的两个方法：**

- notify方法用来监听收到的信号
- stop方法用来取消监听

### 监听全部信号

```go
func main() {
	ch := make(chan os.Signal)
	// 监听全部信号
	signal.Notify(ch)
	fmt.Println("启动了程序")
	s := <-ch
	fmt.Println("收到信号：", s)
}
```

### 监听指定信号

```go
func main() {
	ch := make(chan os.Signal)
	//signal.Notify(ch, os.Interrupt, os.Kill, syscall.SIGUSR1, syscall.SIGUSR2)
	signal.Notify(ch, os.Interrupt, os.Kill)
	fmt.Println("启动程序了")
	s := <-ch
	fmt.Println("收到信号：", s) // 收到信号： interrupt
}
```

### 优雅退出信号

```go
// 优雅退出信号
func waitElegantExit(signalChan chan os.Signal) {
	for i := range signalChan {
		switch i {
		case syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT:
			// 这里做一些清理操作或退出相关操作, 如：关闭数据库
			fmt.Println("接收到退出信号：", i.String())
			os.Exit(0)
		}
	}
}

// 优雅退出
func main() {
	ch := make(chan os.Signal)
	/*
		SIGHUP：1 终端控制进程结束（终端控制断开）
		SIGINT：2 用户发送INTR字符（ctrl+c）触发
		SIGTERM：15 结束程序（可以被捕获，阻塞或忽略）
		SIGQUIT：用户发送QUIT字符（ctrl+\）
	*/
	signal.Notify(ch, syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT)

	// 阻塞，直到接收到退出信号后才停止进程
	waitElegantExit(ch)
}
```

### 优雅退出信号(封装)

```go
func NewShutdownSignal() chan os.Signal {
	ch := make(chan os.Signal)
	signal.Notify(ch, syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT)
	return ch
}
func WaitExit(ch chan os.Signal, exit func()) {
	for i := range ch {
		switch i {
		case syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT:
			fmt.Println("接收到退出信号：", i.String())
			// 退出的一些操作
			exit()
			os.Exit(0)
		}
	}
}
// 优雅退出
func main() {
	signalChan := NewShutdownSignal()
	WaitExit(signalChan, func() {
		fmt.Println("退出操作")
	})
}
```



