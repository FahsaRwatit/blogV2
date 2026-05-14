---
title: "Go标准库中的Cond"
description: "Go标准库中的Cond"
date: 2022-08-16 18:10:00
slug: Golang-Cond
image:
categories:
    - Go
tags: ["Go"]
---

### 使用Cond的示例

10 个运动员进入赛场之后需要先做拉伸活动活动筋骨，向观众和粉丝招手致敬，在自己的赛道上做好准备；
等所有的运动员都准备好之后，裁判员才会打响发令枪。

```go
import (
	"log"
	"math/rand"
	"sync"
	"time"
)
/*
10 个运动员进入赛场之后需要先做拉伸活动活动筋骨，向观众和粉丝招手致敬，在自己的赛道上做好准备；
等所有的运动员都准备好之后，裁判员才会打响发令枪。
*/
func main() {
	c := sync.NewCond(&sync.Mutex{})
	var ready int
	for i := 0; i < 10; i++ {
		go func(i int) {
			time.Sleep(time.Duration(rand.Int63n(10)) * time.Second)
			// 加锁更改等待条件
			c.L.Lock()
			ready++
			c.L.Unlock()

			log.Printf("运动员#%d:已准备就绪\n", i)
			c.Broadcast()
		}(i)
	}
	c.L.Lock()
	for ready != 10 {
		c.Wait()
		log.Println("裁判员被唤醒一次")
	}
	c.L.Unlock()
	log.Println("所有的运动员都准备就绪，比赛开始3,2,1...")
}
```

### Cond基本用法

Cond 并发原语初始化时，需要关联一个 Locker 接口的实例，一般我们使用 Mutex 或者 RWMutex。

```go
type Cond struct {}
func NewCond(l Locker) *Cond {}
func (c *Cond) Wait() {}
func (c *Cond) Signal() {}
func (c *Cond) Broadcast() {}
```

**Signal**：允许调用者 Caller 唤醒一个等待此 Cond 的 goroutine

**Broadcast**：允许调用者 Caller 唤醒所有等待此 Cond 的 goroutine。

**Wait**：会把调用者 Caller 放入 Cond 的等待队列中并阻塞，直到被 Signal 或者 Broadcast 的方法从等待队列中移除并唤醒。*调用Wait必须要持有c.L的锁*

### Cond的实现原理

sync/cond.go文件下：

```go
type Cond struct {
	noCopy noCopy

	// L is held while observing or changing the condition
    // 当观察或者修改等待条件的时候需要加锁
	L Locker
	// 等待队列
	notify  notifyList
	checker copyChecker
}
// NewCond returns a new Cond with Locker l.
func NewCond(l Locker) *Cond {
	return &Cond{L: l}
}

//	... make use of condition ...
//	c.L.Unlock()
func (c *Cond) Wait() {
	c.checker.check()
    // 增加到等待队列中
	t := runtime_notifyListAdd(&c.notify)
	c.L.Unlock()
    // 阻塞休眠直到被唤醒
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock()
}

// Signal wakes one goroutine waiting on c, if there is any.
//
// It is allowed but not required for the caller to hold c.L
// during the call.
//
// Signal() does not affect goroutine scheduling priority; if other goroutines
// are attempting to lock c.L, they may be awoken before a "waiting" goroutine.
func (c *Cond) Signal() {
	c.checker.check()
	runtime_notifyListNotifyOne(&c.notify)
}

// Broadcast wakes all goroutines waiting on c.
//
// It is allowed but not required for the caller to hold c.L
// during the call.
func (c *Cond) Broadcast() {
	c.checker.check()
	runtime_notifyListNotifyAll(&c.notify)
}
```

`runtime_notifyListXXX`：是运行时实现的方法，实现一个等待通知的队列。详见`runtime/sema.go`

`copyChecker`：是辅助结构，可以在运行时检查Cond是否被复制使用

Signal 和 Broadcast 只涉及到 notifyList 数据结构，不涉及到锁。

Wait 把调用者加入到等待队列时会释放锁，在被唤醒之后还会请求锁。在阻塞休眠期间，调用者是不持有锁的，这样能让其他 goroutine 有机会检查或者更新等待变量。

### 使用Cond常见的2个错误

- 调用Wait的时候没有加锁
- 没有检查条件是否满足情况就继续执行了

#### 调用Wait的时候没有加锁

将示例中的锁去除，会panic

```go
// c.L.Lock()
for ready != 10 {
    c.Wait()
    log.Println("裁判员被唤醒一次")
}
// c.L.Unlock()

// fatal error: sync: unlock of unlocked mutex
```

#### 没有检查条件是否满足情况就继续执行了

waiter groutine 被唤醒不等于条件被满足

```go
c.L.Lock()
// for ready != 10 {
    c.Wait()
    log.Println("裁判员被唤醒一次")
// }
c.L.Unlock()
// 条件未满足程序就执行完了
```