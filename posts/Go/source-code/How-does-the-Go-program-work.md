---
title: "Go程序是怎么运行的？"
description: "Go程序是怎么运行的？"
date: 2022-10-08 18:10:00
slug: How-does-the-Go-program-work
image:
categories:
    - Go
tags: ["Go","Go底层"]
---



### 关于Go程序的入口？

Go 程序的入口不是main方法，是runtime下的rt0_XXX.s，runtime/rt0_XXX.s

rt0_XXX.s 不是一个Go程序，是一个汇编语言

### 读取命令行参数

复制参数数量argc和参数值argv到栈上

### 初始化 g0 执行栈

g0 是为了调度协程而产生的协程

g0 是每个Go程序的第一个协程

### 运行时检测

- 检查各种类型的长度
- 检查指针操作
- 检查结构体字段的偏移量
- 检查atomic原子操作
- 检查CAS操作
- 检查栈大小是否是2的幂次

### 初始化 runtime.args

- 对命令行中的参数进行处理
- 参数数量赋值给argc int32
- 参数值复制给argv **byte

### 调度器初始化 runtime.schedinit

- 全局栈空间内存分配
- 加载命令行参数到os.Args
- 堆内存空间的初始化
- 加载操作系统环境变量
- 初始化当前系统线程
- 垃圾回收器的参数初始化
- 算法初始化（map、hash）
- 设置process数量

### 创建主协程

- 创建一个新的协程，执行runtime.main
- 放入调度器等待调度

### 初始化M

- 初始化一个M，用来调度主协程

### 主协程执行主函数

- 执行runtime包中的init方法
- 启动GC垃圾收集器
- 执行用户包依赖的init方法
- 执行用户包依赖的init方法
- 执行用户主函数main.main()

### 总结

- Go启动时经历了检查、各种初始化、初始化协程调度的过程
- main.main()也是在协程中运行的

