---
title: "Golang中的RWMutex"
description:  "Golang中的RWMutex"
date: 2022-09-18 18:10:00
slug: RWMutex-in-Golang
image:
categories:
    - Go
tags: ["Golang"]
---

### RWMutex实现线程安全的计数器

**Lock/Unlock**：写操作时调用的方法。如果锁已经被 reader 或者 writer 持有，那么，Lock 方法会一直阻塞，直到能获取到锁；Unlock 则是配对的释放锁的方法。

**RLock/RUnlock**：读操作时调用的方法。如果锁已经被 writer 持有的话，RLock 方法会一直阻塞，直到能获取到锁，否则就直接返回；而 RUnlock 是 reader 释放锁的方法。

**RLocker**：这个方法的作用是为读操作返回一个 Locker 接口的对象。它的 Lock 方法会调用 RWMutex 的 RLock 方法，它的 Unlock 方法会调用 RWMutex 的 RUnlock 方法。

如果你遇到可以明确区分 reader 和 writer goroutine 的场景，且有大量的并发读、少量的并发写，并且有强烈的性能需求，你就可以考虑使用读写锁 RWMutex 替换 Mutex。

```go
type Counter struct {
	mu    sync.RWMutex
	count uint64
}
// 使用写锁
func (c *Counter) Incr() {
	c.mu.Lock()
	c.count++
	c.mu.Unlock()
}
// 使用读锁
func (c *Counter) Count() uint64 {
	c.mu.RLock()
	defer c.mu.RUnlock()
	return c.count
}
func main() {
	var counter Counter
	// 10个reader
	for i := 0; i < 10; i++ {
		go func() {
			for {
				fmt.Println(counter.Count())
				time.Sleep(time.Microsecond)
			}
		}()
	}
	// 1个writer
	for {
		// 计数器写操作
		counter.Incr()
		time.Sleep(time.Second)
	}
}
```

### RWMutex实现原理

Go 标准库中的 RWMutex 是基于 Mutex 实现的。

readers-writers 问题有三类，读写锁的设计和实现也分成三类：

- Read-preferring
- Write-preferring
- 不指定优先级

Go 标准库中的 RWMutex 设计是 Write-preferring 方案。一个正在阻塞的 Lock 调用会排除新的 reader 请求到锁。

源代码：sync/rwmutex.go

```go
type RWMutex struct {
	w           Mutex  // 互斥锁，解决多个writer的竞争
	writerSem   uint32 // writer 信号量
	readerSem   uint32 // reader 信号量
	readerCount int32  // reader 的数量
	readerWait  int32  // writer等待完成的reader数量
}
const rwmutexMaxReaders = 1 << 30  // 最大的reader数量
```

- 字段 w：为 writer 的竞争锁而设计；
- 字段 readerCount：记录当前 reader 的数量（以及是否有 writer 竞争锁）；
- readerWait：记录 writer 请求锁时需要等待 read 完成的 reader 的数量；
- writerSem 和 readerSem：都是为了阻塞设计的信号量。

### RLock/RUnlock 的实现

移除了`race`等无关紧要的代码。

`readerCount`：有可能为正数，也有可能为负数，有双重含义：

- 没有 writer 竞争或持有锁时，readerCount 和我们正常理解的 reader 的计数是一样的；
- 但是，如果有 writer 竞争锁或者持有锁时，那么，readerCount 不仅仅承担着 reader 的计数功能，还能够标识当前是否有 writer 竞争或持有锁。

```go
func (rw *RWMutex) RLock() {
    // 对reader计数加1
    // readerCount为负数代表有等待中的writer或持有锁的writer,
    if atomic.AddInt32(&rw.readerCount, 1) < 0 {
		// A writer is pending, wait for it.
        // rw.readerCount是负值的时候说明这时有writer等待请求锁
        // 因为writer的优先级高，所以把后来的reader阻塞休眠
		runtime_SemacquireMutex(&rw.readerSem, false, 0)
	}
}

func (rw *RWMutex) RUnlock() {
    // 对reader计数减1
    if r := atomic.AddInt32(&rw.readerCount, -1); r < 0 {
		// Outlined slow-path to allow the fast-path to be inlined
        // 有等待的writer
		rw.rUnlockSlow(r)
	}
}
func (rw *RWMutex) rUnlockSlow(r int32) {
    // A writer is pending.
	if atomic.AddInt32(&rw.readerWait, -1) == 0 {
		// The last reader unblocks the writer.
        // 最后一个reader解锁writer, writer获得锁
		runtime_Semrelease(&rw.writerSem, false, 1)
	}
}
```

竞争中的Writer会等待所有持有锁的Reader都释放完，才有可能持有这把锁。

### Lock

RWMutex 是一个多 writer 多 reader 的读写锁。为了避免writer之间竞争。RWMutex使用用Mutex来保证互斥。

移除了`race`等无关紧要的代码。

```go
// Lock locks rw for writing.
// If the lock is already locked for reading or writing,
// Lock blocks until the lock is available.
func (rw *RWMutex) Lock() {
    // First, resolve competition with other writers.
    // 首先解决其他writer竞争的问题
	rw.w.Lock()
	// Announce to readers there is a pending writer.
    // 反转readerCount，告诉reader有writer竞争锁
	r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) + rwmutexMaxReaders
	// Wait for active readers.
    // 如果当前有reader持有锁，那么需要等待
	if r != 0 && atomic.AddInt32(&rw.readerWait, r) != 0 {
		runtime_SemacquireMutex(&rw.writerSem, false, 0)
	}
}
```

一旦一个 writer 获得了内部的互斥锁，就会反转 readerCount 字段，把它从原来的正整数 readerCount(>=0) 修改为负数（readerCount-rwmutexMaxReaders），让这个字段保持两个含义（既保存了 reader 的数量，又表示当前有 writer）。

### Unlock

移除了`race`等无关紧要的代码。

```go
func (rw *RWMutex) Unlock() {
    // Announce to readers there is no active writer.
    // 反转readerCount加上rwmutexMaxReaders，为正数
    // 告诉reader没有活跃的writer了
	r := atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders)
    // Unblock blocked readers, if any.
    // 唤醒新来的阻塞的reader
	for i := 0; i < int(r); i++ {
		runtime_Semrelease(&rw.readerSem, false, 0)
	}
	// Allow other writers to proceed.
    // 释放内部的互斥锁
	rw.w.Unlock()
}
```

注意 ：

- readerCount的含义和反转方式
- Lock方法中先获取锁，再执行其他代码
- Unlock方法中先执行其他代码，最后才释放锁 

### RWMutex的三个踩坑点

- 1、不可复制
- 2、重入导致死锁
- 3、释放未加锁的RWMutex

`kubernetes的issue 62464`：https://github.com/kubernetes/kubernetes/pull/62464