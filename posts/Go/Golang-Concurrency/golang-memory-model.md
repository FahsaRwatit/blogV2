---
title: "Golang的内存模型保证并发读写的顺序"
description: "Golang的内存模型保证并发读写的顺序"
date: 2022-08-20 18:10:00
slug: golang-memory-model
image:
categories:
    - Go
tags: ["Go"]
---

### Golang中的happens-before关系

### init函数

- 先传递给 Go 编译器的源文件中的 init 函数，会先被执行。
- main函数一定在导入的包的init函数之后执行
- init()函数不能手工显示调用。
- 依赖包按照“深度优先”的顺序进行初始化
- 包内按照：常量 -> 变量 -> init 函数
- 包内的多个 init 函数按出现次序进行自动调用。（同一个包下的多个文件，会按照文件名的排列顺序进行初始化。）
- 同一个源文件中的多个 init 函数，会按声明顺序依次执行

#### 示例

```go
文件结构：
-p1
	-lib1.go
-p2
	-lib1.go
-p3
	-lib1.go
	-lib2.go
-trace
	lib1.go
main.go
```

`包trace`：

```go
func Trace(t string, v int) int {
	fmt.Println(t, ":", v)
	return v
}
```

`包p3`

```go
// p3/lib1.go

var V1_p3 = trace.Trace("init_v1_p3:", 3)
var V2_p3 = trace.Trace("init_v2_p3:", 3)

func init() {
	fmt.Println("init func in p3")
	V1_p3 = 300
	V2_p3 = 300
}

// p3/lib2.go

var V3_p3 = trace.Trace("init_v3_p3:", 5)

func init() {
	fmt.Println("Another init func in p3")
}
```

`包p2`：

```go
var V1_p2 = trace.Trace("init v1_p2:", 2)
var V2_p2 = trace.Trace("init v2_p2:", p3.V2_p3)

func init() {
	fmt.Println("init func in p2")
	V1_p2 = 200
}
```

`包p1`：

```go
var V1_p1 = trace.Trace("init_v1_p1:", p2.V1_p2)
var V2_p1 = trace.Trace("init_v2_p1:", p2.V2_p2)

func init() {
	fmt.Println("init func in p1")
}
```

`main.go`：

main程序中，依赖包p1，包p1依赖p2，p2依赖p3

```go
func init() {
	fmt.Println("init func in main")
	fmt.Println(c1)
}

var V1_main = trace.Trace("init_v1_main:", 88)

const c1 = 35

// main函数一定在导入的包的init函数之后执行
func main() {
	fmt.Println("V1_p1:", p1.V1_p1)
	fmt.Println("V2_p1:", p1.V2_p1)
}

/*
执行输出：
init_v1_p3: : 3
init_v2_p3: : 3
init_v3_p3: : 5
init func in p3
Another init func in p3
init v1_p2: : 2
init v2_p2: : 300
init func in p2
init_v1_p1: : 200
init_v2_p1: : 300
init func in p1
init_v1_main: : 88
35
init func in main
V1_p1: 200
V2_p1: 300
*/
```

### goroutine

启动 goroutine 的 go 语句的执行，一定 happens before 此 goroutine 内的代码执行。

```go
var a string

func f() {
	fmt.Println(a)
}
func main() {
	a = "hello golang"
	go f()
	time.Sleep(1 * time.Second)
}
// 无论如何，f()函数中输出的a都为"hello golang"
```

### Channel

- 往 Channel 中的发送操作，happens before 从该 Channel 接收相应数据的动作完成之前，即第 n 个 send 一定 happens before 第 n 个 receive 的完成。

```go
func f() {
	s = "hello golang"
	ch <- struct{}{}
}

var ch = make(chan struct{}, 10)
var s string

func main() {
	go f()
	<-ch
	fmt.Println(s)
}
```

- close 一个 Channel 的调用，肯定 happens before 从关闭的 Channel 中读取出一个零值。
- 对于 unbuffered 的 Channel，也就是容量是 0 的 Channel，从此 Channel 中读取数据的调用一定 happens before 往此 Channel 发送数据的调用完成。

```go
var a string
var ch = make(chan struct{})

func f() {
	a = "hello golang"
	<-ch
}
func main() {
	go f()
	ch <- struct{}{}
	fmt.Println(a)
}
```

- 如果 Channel 的容量是 m（m>0），那么，第 n 个 receive 一定 happens before 第 n+m 个 send 的完成。

### Mutex/RWMutex

读写锁的 Lock 必须等待既有的读锁释放后才能获取到。

```go
var mu sync.Mutex
var s string
func f() {
	s = "hello golang"
	mu.Unlock()
}
func main() {
	mu.Lock()
	go f()
	mu.Lock() // 一定在f()中的mu.Unlock()后执行
	fmt.Println(s)
}
```

### WaitGroup

`Wait`方法等到计数值归零后才返回。

### Once

对于 once.Do(f) 调用，f 函数的那个单次调用一定 happens before 任何 once.Do(f) 调用的返回.

换句话说，就是函数 f 一定会在 Do 方法返回之前执行。

```go
var s string
var once sync.Once

func f() {
	s = "helllo golang"
}
func main() {
	once.Do(f) // 函数返回时，s一定会被初始化
	fmt.Println(s)
}
```

### atomic

没有明确给出`atomic`的保证，现阶段不要使用`atomic`来保证顺序性。

```go
func main() {
	var a, b int32 = 0, 0
	go func() {
		atomic.StoreInt32(&a, 1)
		atomic.StoreInt32(&b, 1)
	}()
	for atomic.LoadInt32(&b) == 0 {
		// 暂停当前goroutine，使其他goroutine先行运算。只是暂停，不是挂起，当时间片轮转到该协程时，Gosched()后面的操作将自动恢复
		runtime.Gosched()
	}
	fmt.Println(atomic.LoadInt32(&a))
}
```

`If you must read the rest of this document to understand the behavior of your program, you are being too clever.`

`Don’t be clever.`

谨慎使用，但是不要以为完全理解，否则很容易掉进`坑`里去。
