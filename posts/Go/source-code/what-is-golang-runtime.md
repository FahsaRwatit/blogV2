---
title: "关于GoLang的Runtime"
description: "关于GoLang的Runtime"
date: 2021-10-08 18:10:00
slug: what-is-golang-runtime
image:
categories:
    - Go
tags: ["Go","Go底层"]
---

`Runtime`:翻译过来是`运行时`,在 Golang 的源码包中我们可以找到它。

指的是程序在运行时作为支撑的部分，Runtime就是程序的运行环境。

Runtime并不是一个新的概念，在其他语言中也可见，例如Java的虚拟机，支撑Java在运行时的组件。也例如JavaScript的浏览器内核。

### Golang的Runtime特点

- 没有虚拟机的概念
- Runtime作为程序的一部分打包进二进制产物
- Runtime随用户程序一起运行 （说白了就是Runtime也是一堆的Go代码或者底层的汇编代码和我们写的程序没明显区别）
- Runtime与用户程序没有明显界限，直接通过函数调用

也就是 Code + Runtime 打包成 Binary 来运行。

### Golang的Runtime能力

- 内存管理能力 （Runtime可以帮助我们去分配内存，比如堆内存、栈内存及我们的变量分配到哪里等）

- 垃圾回收的能力（GC）

- 超强的并发能力（协调调度）

- Runtime有一定的屏蔽系统调用能力（为了做到跨平台，Runtime会处理各个系统的不同调用）

- 一些Go的关键字其实就是Runtime下的函数：

  | 关键字 | 函数                            |
  | ------ | ------------------------------- |
  | go     | newproc                         |
  | new    | newobject                       |
  | make   | makeslice, makechain,makemap... |
  | <-     | chansend1, chanrecv1            |

  ### 总结

  - Go 的Runtime负责内存管理、垃圾回收、协调调度
  - Go 的 Runtime被编译为用户程序的一部分、一起运行