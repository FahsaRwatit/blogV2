---
title: "Golang的Mutex容易出错的四大场景"
description: "Golang的Mutex容易出错的四大场景"
date: 2022-09-10 18:10:00
slug: "Golang-Mutex-Four-Error-Prone-Scenarios"
image:
categories:
    - Go
tags: ["Go","Mutex"]
---

###常见的四种错误场景

- Lock/Unlock不是成对出现
- Copy已经使用的Mutex
- 重入
- 死锁

### Lock/Unlock不是成对出现

- 代码中有太多的 if-else 分支，可能在某个分支中漏写了 Unlock；
- 在重构的时候把 Unlock 给删除了；
- Unlock 误写成了 Lock。
- 误删除了Lock。

### Copy已经使用的Mutex

`Tips`：Package sync 的同步原语在使用后是不能复制的

可以使用vet工具检测mutex复制问题

Copy已使用的Mutex的示例：

```go
import (
	"fmt"
	"sync"
)
// Copy已使用的Mutex的示例
// 可以使用vet工具检测mutex复制问题
type Counter struct {
	sync.Mutex
	Count int
}
// 通过复制的方式传入
func foo(c Counter) {
	c.Lock()
	defer c.Unlock()
	fmt.Println("in foo")
}
func main() {
	var c Counter
	c.Lock()
	defer c.Unlock()
	c.Count++
	// 复制锁
	foo(c)
}
/*
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [semacquire]:
sync.runtime_SemacquireMutex(0xc00000e0c0?, 0x0?, 0xc000028000?)
        D:/mysoft/Go/src/runtime/sema.go:77 +0x25
sync.(*Mutex).lockSlow(0xc00000e0c0)
        D:/mysoft/Go/src/sync/mutex.go:171 +0x165
sync.(*Mutex).Lock(...)
        D:/mysoft/Go/src/sync/mutex.go:90
main.foo({{0x1, 0x0}, 0x1})
        D:/myproject/1-pro/go-test/go-subject/concurrency/mutex2_1.go:15 +0x71
main.main()
        D:/myproject/1-pro/go-test/go-subject/concurrency/mutex2_1.go:26 +0x9b
exit status 2
*/

```

### 重入

什么是可重入的锁？
当一个线程获取锁时，如果没有其它线程拥有这个锁，那么，这个线程就成功获取到这个锁。
之后，如果其它线程再请求这个锁，就会处于阻塞等待的状态。
但是，如果拥有这把锁的线程再请求这把锁的话，不会阻塞，而是成功返回，所以叫可重入锁（有时候也叫做递归锁）。
只要你拥有这把锁，你可以可着劲儿地调用，比如通过递归实现一些算法，调用者不会阻塞或者死锁。

**Golang的Mutex不是可重入的锁**

误用Mutex的重入示例：

```go
import (
	"fmt"
	"sync"
)

// 误用Mutex的重入示例
// Mutex不是可重入的锁
/*
当一个线程获取锁时，如果没有其它线程拥有这个锁，那么，这个线程就成功获取到这个锁。
之后，如果其它线程再请求这个锁，就会处于阻塞等待的状态。
但是，如果拥有这把锁的线程再请求这把锁的话，不会阻塞，而是成功返回，所以叫可重入锁（有时候也叫做递归锁）。
只要你拥有这把锁，你可以可着劲儿地调用，比如通过递归实现一些算法，调用者不会阻塞或者死锁。
*/
func foo(l sync.Locker) {
	fmt.Println("in foo")
	l.Lock()
	bar(l)
	l.Unlock()
}

func bar(l sync.Locker) {
	l.Lock()
	fmt.Println(" in bar")
	l.Unlock()
}

func main() {
	l := &sync.Mutex{}
	foo(l)
}

// in foo
// fatal error: all goroutines are asleep - deadlock!
```

**实现重入锁的两种方案：**

- 方案一：通过 hacker 的方式获取到 goroutine id，记录下获取锁的 goroutine id，它可以实现 Locker 接口。
- 方案二：调用 Lock/Unlock 方法时，由 goroutine 提供一个 token，用来标识它自己，而不是我们通过 hacker 的方式获取到 goroutine id，但是，这样一来，就不满足 Locker 接口了。

**goroutine id 实现**

实现可以使用的可重入锁

```go
import (
	"fmt"
	"sync"
	"sync/atomic"
	"github.com/petermattis/goid"
)
// RecursiveMutex包装一个Mutex实现可重入
type RecursiveMutex struct {
	sync.Mutex
	owner     int64 //当前持有锁的goroutine id
	recursion int32 // 这个goroutine 重入的次数
}

// petermattis/goid
func (m *RecursiveMutex) Lock() {
	gid := goid.Get()
	// 如果当前持有锁的goroutine就是这次调用的gorutine，说明是重入的
	if atomic.LoadInt64(&m.owner) == gid {
		m.recursion++
		return
	}
	m.Mutex.Lock()
	// 获得锁的goroutine第一次调用，记录下它的goroutine id,调用次数加1
	atomic.StoreInt64(&m.owner, gid)
	m.recursion = 1
}

func (m *RecursiveMutex) Unlock() {
	gid := goid.Get()
	// 非持有锁的goroutine尝试释放锁，错误使用
	if atomic.LoadInt64(&m.owner) != gid {
		panic(fmt.Sprintf("wrong the ower(%d): %d!", m.owner, gid))
	}
	// 调用次数减1
	// 如果这个goroutine还没有完全释放，则完全返回
	if m.recursion != 0 {
		return
	}
	// 此时 goroutine最后一次 调用 需要释放锁
	atomic.StoreInt64(&m.owner, -1)
	m.Mutex.Unlock()
}

func foo(l *RecursiveMutex) {
	fmt.Println("in foo")
	l.Lock()
	bar(l)
	l.Unlock()
}

func bar(l *RecursiveMutex) {
	l.Lock()
	fmt.Println("in bar")
	l.Unlock()
}

func main() {
	l := &RecursiveMutex{}
	foo(l)
}
// in foo
// in bar
```

**token 实现**

```go
import (
	"fmt"
	"sync"
	"sync/atomic"
)

/*
调用者自己提供token 获取锁的时候，把这个token传入,释放锁的时候也需要将这个token传入。
通过用户传入的token替换前一方案的goroutine id，其实逻辑是一样的
*/

// Token方式的递归锁
type TokenRecursiveMutex struct {
	sync.Mutex
	token     int64
	recursive int32
}

// 请求锁 需要传入token
func (m *TokenRecursiveMutex) Lock(token int64) {
	// 如果传入的token和持有锁的token是一致的，说明是重入的
	if atomic.LoadInt64(&m.token) == token {
		m.recursive++
		return
	}
	// 传入的token不一致，说明不是递归调用
	m.Mutex.Lock()
	// 抢到锁后记录这个token
	atomic.StoreInt64(&m.token, token)
	m.recursive = 1
}

// 释放锁
func (m *TokenRecursiveMutex) Unlock(token int64) {
	if atomic.LoadInt64(&m.token) != token {
		// 释放其他token持有的锁 报错
		panic(fmt.Sprintf("wrong the owner(%d): %d!", m.token, token))
	}
	// 当前持有这个锁的token释放锁
	m.recursive--
	// 还没有回退到最初的 递归调用
	if m.recursive != 0 {
		return
	}
	// 没有递归了，释放锁
	atomic.StoreInt64(&m.token, 0)
	m.Mutex.Unlock()
}

func foo(l *TokenRecursiveMutex) {
	fmt.Println("token: in foo")
	l.Lock(token)
	bar(l)
	l.Unlock(token)
}

func bar(l *TokenRecursiveMutex) {
	l.Lock(token)
	fmt.Println("token: in bar")
	l.Unlock(token)
}

var token int64

func main() {
	token = 1282
	l := &TokenRecursiveMutex{}
	foo(l)
}

// token: in foo
// token: in bar
```

### 死锁

什么是死锁？
两个或两个以上的进程（或线程，goroutine）在执行过程中，因争夺共享资源而处于一种互相等待的状态，如果没有外部干涉，它们都将无法推进下去，此时，我们称系统处于死锁状态或系统产生了死锁。

死锁产生的必要条件？

避免死锁只要破坏其中的一个或几个即可。

1、互斥：至少一个资源是排他性独享的，其他线程必须处于等待状态，直到资源被释放。

2、等待和持有：goroutine持有一个资源，并且还在请求其他goroutine持有的资源。

3、不可剥夺：资源只能由持有它的goroutine来释放

4、环路等待：存在一组等待进程P1~PN，P1等待P2持有的资源，P2等待P3持有的资源，以此类推，最后PN等待 P1持有的资源，这就形成了一个环路等待的死结。

### 总结

1、保证 Lock/Unlock 成对出现，尽可能采用 defer mutex.Unlock 的方式，把它们成对、紧凑地写在一起。

2、手误和重入导致的死锁，是最常见的使用 Mutex 的 Bug

### 流行的项目踩坑

- https://github.com/kubernetes/kubernetes/pull/72361
- https://github.com/kubernetes/kubernetes/pull/45192