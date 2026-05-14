---
title: "Golang的sync.Pool"
description: "Golang的sync.Pool"
date: 2022-09-15 18:10:00
slug: Golang-sync-pool
image:
categories:
    - Go
tags: ["Go"]

---

Go是自动垃圾回收的编程语言，采用三色并发标记算法，标记对象并回收。如果你想使用 Go 开发一个高性能的应用程序的话，就必须考虑垃圾回收给性能带来的影响。

### sync.Pool

Go标准库提供的sync.Pool的数据类型，用来保存一组可独立访问的**临时**对象的池子。

注意：如果池子中的对象没有被别的对象引用的话，未来就会被垃圾回收掉。

#### 关于Pool的两个知识点：

- sync.Pool 本身就是线程安全的，多个 goroutine 可以并发地调用它的方法存取对象；
- sync.Pool 不可在使用之后再复制使用。

### sync.Pool使用

```go
var pool *sync.Pool
type Person struct {
	Name string
}
func initPool() {
	pool = &sync.Pool{
		New: func() interface{} {
			fmt.Println("Create a Person")
			return new(Person)
		},
	}
}
func main() {
	initPool()
	p := pool.Get().(*Person)
	fmt.Println("首次从pool中获取:", p) // 首次从pool中获取: &{}
	p.Name = "Mario"
	fmt.Printf("设置p.Name=%s\n", p.Name) // 设置p.Name=Mario

	pool.Put(p)
	fmt.Println("pool中有对象，使用get获取:", pool.Get().(*Person)) // pool中有对象，使用get获取: &{Mario}
	fmt.Println("无对象，使用get获取:", pool.Get().(*Person))      //  无对象，使用get获取: &{}
}
```

### sync.Pool的方法

`sync.Pool`对外提供了三个方法：New、Get、Put

#### New

```go
New func() any
```

New方法用来创建新的元素，如果没有设置New，当前对象池中没有可用的元素时，调用Get方法将返回nil.



















