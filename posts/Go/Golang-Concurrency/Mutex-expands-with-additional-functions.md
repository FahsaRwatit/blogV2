---
title: "Mutex拓展额外功能"
description: "Mutex拓展额外功能-TryLock-获取等待者信息-线程安全的队列"
date: 2022-09-10 18:10:00
slug: Mutex-expands-with-additional-functions
image:
categories:
    - Go
tags: ["Go"]

---

### 补充

**iota的5点：**

- 不同 const 定义块互不干扰
- 所有注释行和空行全部忽略
- 没有表达式的常量定义复用上一行的表达式
- 从第一行开始，iota 从 0 逐行加一
- 替换所有 iota

参考文章：https://studygolang.com/articles/22468?fr=sidebar

### 基于Mutex实现 TryLock 方法

- 1 << 0  是把1 按2进制 左移0位，结果还是 1 ,2进制 0000 0001
- 1 << 1, 是把1 按2进制 左移1位，结果是2,2进制 0000 0010

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"sync/atomic"
	"time"
	"unsafe"
)

// 基于Mutex实现 TryLock 方法

// 复制Mutex定义的常量
// 1 << 0  是把1 按2进制 左移0位，结果还是 1 ,2进制 0000 0001
// 1 << 1, 是把1 按2进制 左移1位，结果是2,2进制 0000 0010
// &	与	两个位都为1时，结果才为1
// 或	两个位都为0时，结果才为0
const (
	mutexLocked      = 1 << iota // 加锁标识位置 1  1 << 0   => 0001
	mutexWoken                   // 唤醒标识位置 2  1 << 1   => 0010
	mutexStarving                //锁饥饿标识位置 4  1 << 2  => 0100
	mutexWaiterShift = iota      // 标识waiter的起始位置 3
)

// 拓展一个Mutex结构
type Mutex struct {
	sync.Mutex
}

// 尝试获取锁
// 如果addr和old相同,就用new代替addr ok:=atomic.CompareAndSwapInt32(&addr,old,new)
func (m *Mutex) TryLock() bool {
	// 如果成功抢到锁
	if atomic.CompareAndSwapInt32((*int32)(unsafe.Pointer(&m.Mutex)), 0, mutexLocked) {
		return true
	}
	// 如果处于唤醒、加锁或者饥饿状态，这次请求就不参与竞争了，返回false
	// 如果锁已经被其他 goroutine 所持有，或者被其他唤醒的 goroutine 准备持有，那么，就直接返回 false，不再请求
	old := atomic.LoadInt32((*int32)(unsafe.Pointer(&m.Mutex)))
	if old&(mutexLocked|mutexStarving|mutexWoken) != 0 {
		return false
	}
	// 尝试在竞争的状态下请求锁
	new := old | mutexLocked
	return atomic.CompareAndSwapInt32((*int32)(unsafe.Pointer(&m.Mutex)), old, new)
}

// 测试TryLock机制是否工作
func try() {
	var mu Mutex
	// 启动一个goroutine持有一段时间的锁
	go func() {
		mu.Lock()
		time.Sleep(time.Duration(rand.Intn(2)) * time.Second)
		mu.Unlock()
	}()
	time.Sleep(time.Second)
	// 尝试获取到锁
	ok := mu.TryLock()
	// 获取成功
	if ok {
		fmt.Println("go the lock")
		mu.Unlock()
		return
	}
	// 没有获取到锁
	fmt.Println("cant't get the lock")
}

func main() {
	fmt.Println(mutexLocked)
	fmt.Println(mutexWoken)
	fmt.Println(mutexStarving)
	fmt.Println(mutexWaiterShift)
	try()
}

```

### 获取等待者信息

Mutex 结构中的 state 字段有很多个含义，通过 state 字段，你可以知道锁是否已经被某个 goroutine 持有、当前是否处于饥饿状态、是否有等待的 goroutine 被唤醒、等待者的数量等信息。但是，state 这个字段并没有暴露出来。

- state 这个字段的第一位是用来标记锁是否被持有，
- 第二位用来标记是否已经唤醒了一个等待者，
- 第三位标记锁是否处于饥饿状态，

通过分析这个 state 字段我们就可以得到这些状态信息。
我们可以为这些状态提供查询的方法，这样就可以实时地知道锁的状态了。

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
	"time"
	"unsafe"
)

// 获取等待者数量等指标

/*
Mutex 结构中的 state 字段有很多个含义，
通过 state 字段，你可以知道锁是否已经被某个 goroutine 持有、当前是否处于饥饿状态、
是否有等待的 goroutine 被唤醒、等待者的数量等信息。但是，state 这个字段并没有暴露出来。
type Mutex struct {
	state int32
	sema uint32
}
可以通过unsafe的方式获取state字段，并解析出来
*/

/*
state 这个字段的第一位是用来标记锁是否被持有，
第二位用来标记是否已经唤醒了一个等待者，
第三位标记锁是否处于饥饿状态，
通过分析这个 state 字段我们就可以得到这些状态信息。
我们可以为这些状态提供查询的方法，这样就可以实时地知道锁的状态了。
*/
const (
	mutexLocked      = 1 << iota // 加锁标识位置 1  1 << 0   => 0001
	mutexWoken                   // 唤醒标识位置 2  1 << 1   => 0010
	mutexStarving                //锁饥饿标识位置 4  1 << 2  => 0100
	mutexWaiterShift = iota      // 标识waiter的起始位置 3
)

// 拓展一个Mutex结构
type Mutex struct {
	sync.Mutex
}

// 获取当前持有和等待这把锁的goroutine的总数
func (m *Mutex) Count() int {
	// 获取state字段的值
	v := atomic.LoadInt32((*int32)(unsafe.Pointer(&m.Mutex)))
	// 得到等待者的数量
	v = v >> mutexWaiterShift
	v = v + (v & mutexLocked) // 再加上锁 持有的数量，0或1
	return int(v)
}

// 锁是否被持有
func (m *Mutex) IsLocked() bool {
	state := atomic.LoadInt32((*int32)(unsafe.Pointer(&m.Mutex)))
	return state&mutexLocked == mutexLocked
}

// 是否有等待者被唤醒
func (m *Mutex) IsWoken() bool {
	state := atomic.LoadInt32((*int32)(unsafe.Pointer(&m.Mutex)))
	return state&mutexWoken == mutexWoken
}

// 锁是否处于饥饿状态
func (m *Mutex) IsStarving() bool {
	state := atomic.LoadInt32((*int32)(unsafe.Pointer(&m.Mutex)))
	return state&mutexStarving == mutexStarving
}

// 输出1000个goroutin并发访问下，锁的状态信息
func test() {
	var mu Mutex
	for i := 0; i < 1000; i++ {
		go func() {
			mu.Lock()
			time.Sleep(time.Second)
			mu.Unlock()
		}()
	}
	time.Sleep(time.Second)
	fmt.Printf("waitings: %d, isLocked: %t, woken: %t, starving: %t \n", mu.Count(), mu.IsLocked(), mu.IsWoken(), mu.IsStarving())
}

func main() {
	test()
}

// 输出：waitings: 998, isLocked: true, woken: false, starving: false

```

### 使用Mutex实现一个线程安全的队列

Mutex常以非线程安全的数据结构一起，组成一个线程安全的数据结构。

新的数据结构的业务逻辑由原来的数据结构提供， Mutex提供锁机制来保证线程安全。

```go
package main

import (
	"fmt"
	"sync"
	"time"
)
// 使用Mutex实现一个线程安全的队列
//
//	Mutex常以非线程安全的数据结构一起，组成一个线程安全的数据结构。
//
// 新的数据结构的业务逻辑由原来的数据结构提供， Mutex提供锁机制来保证线程安全。
type SliceQueue struct {
	data []interface{}
	mu   sync.Mutex
}

func NewSliceQueue(n int) (q *SliceQueue) {
	return &SliceQueue{data: make([]interface{}, 0, n)}
}

// 把值放在队尾
func (q *SliceQueue) Enqueue(v interface{}) {
	q.mu.Lock()
	q.data = append(q.data, v)
	q.mu.Unlock()
}

// 移除对头并返回
func (q *SliceQueue) Dequeue() interface{} {
	q.mu.Lock()
	if len(q.data) == 0 {
		q.mu.Unlock()
		return nil
	}
	v := q.data[0]
	q.data = q.data[1:]
	q.mu.Unlock()
	return v
}

func main() {
	var queue *SliceQueue
	queue = NewSliceQueue(5)
	for i := 0; i < 10; i++ {
		go func(i int) {
			queue.Enqueue(i)
		}(i)
	}
	for i := 0; i < 10; i++ {
		go func(i int) {
			v := queue.Dequeue()
			fmt.Println(v)
		}(i)
	}
	time.Sleep(time.Second)
	fmt.Println("end...")
}

// 运行：go run -race XXX.go
```

