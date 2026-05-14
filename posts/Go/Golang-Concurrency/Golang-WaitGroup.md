---
title: "Golang的WaitGroup"
description: "Golang的WaitGroup"
date: 2022-09-15 18:10:00
slug: Golang-WaitGroup
image:
categories:
    - Go
tags: ["Go"]

---

### 关于WaitGroup的一个示例

```go
import (
	"fmt"
	"sync"
	"time"
)
// 线程安全的计数器
type Counter struct {
	mu    sync.Mutex
	count uint64
}
// 对计数值加1
func (c *Counter) Incr() {
	c.mu.Lock()
	c.count++
	c.mu.Unlock()
}
// 获取当前的计数值
func (c *Counter) Count() uint64 {
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.count
}
// sleep 1秒 然后计数值加1
func worker(c *Counter, wg *sync.WaitGroup) {
	defer wg.Done()
	time.Sleep(time.Second)
	c.Incr()
}
func main() {
	var counter Counter
	var wg sync.WaitGroup
	// WaitGroup的值设置为10
	wg.Add(10)
	for i := 0; i < 10; i++ {
		//启动10个groutine执行任务
		go worker(&counter, &wg)
	}
	// 检查点，等待groutine都完成任务
	wg.Wait()
	// 输出当前计数器的值
	fmt.Println(counter.Count())
}

```

### WaitGroup的实现

**提供了三个方法：**

```go
func (wg *WaitGroup) Add(delta int) {} // 用来设置WaitGroup的计数器
func (wg *WaitGroup) Done() {} // 用来将WaitGroup的计数器数值减1，也就是Add(-1)
func (wg *WaitGroup) Wait() {} // 调用这个方法的 goroutine会一直阻塞，直到WaitGroup的计数值为0
```

WaitGroup的内部实现也陆陆续续改变了好几次,主要是针对它的字段的原子操作不断的做优化。(推荐阅读：https://colobu.com/2022/08/30/waitgroup-to-love-to-toss/)

```go
type WaitGroup struct {
    // 避免复制使用的一个技巧，可以告诉vet工具违反了复制使用的规则
	noCopy noCopy

	// 64-bit value: high 32 bits are counter, low 32 bits are waiter count.
	// 64-bit atomic operations require 64-bit alignment, but 32-bit
	// compilers only guarantee that 64-bit fields are 32-bit aligned.
	// For this reason on 32 bit architectures we need to check in state()
	// if state1 is aligned or not, and dynamically "swap" the field order if
	// needed.
    // 64位值：高32位为计数器，低32位为服务员计数。
	// 64 位原子操作需要 64 位对齐，但 32 位
	// 编译器只保证 64 位字段是 32 位对齐的。
	// 由于这个原因，在 32 位架构上我们需要检查 state()
	// 如果 state1 对齐与否，并在需要时动态“交换”字段顺序。
	state1 uint64
	state2 uint32
}
// state returns pointers to the state and sema fields stored within wg.state*.
// 得到state的地址和信号量的地址
func (wg *WaitGroup) state() (statep *uint64, semap *uint32) {
	if unsafe.Alignof(wg.state1) == 8 || uintptr(unsafe.Pointer(&wg.state1))%8 == 0 {
		// state1 is 64-bit aligned: nothing to do.
		return &wg.state1, &wg.state2
	} else {
		// state1 is 32-bit aligned but not 64-bit aligned: this means that
		// (&state1)+4 is 64-bit aligned.
		state := (*[3]uint32)(unsafe.Pointer(&wg.state1))
		return (*uint64)(unsafe.Pointer(&state[1])), &state[0]
	}
}

```

**Add方法**

去除了`race`调试部分

```go
func (wg *WaitGroup) Add(delta int) {
	statep, semap := wg.state()
    // 高32bit是计数值v,所以把delta左移32，增加到计数上
    // 更新statep,statep将在wait和add中通过原子操作一起使用
	state := atomic.AddUint64(statep, uint64(delta)<<32)
    // 当前计数值
	v := int32(state >> 32)
	w := uint32(state)
    // 计数值v不能为0，否则会panic
	if v < 0 {
		panic("sync: negative WaitGroup counter")
	}
    // wait不等于0说明已经执行了Wait，此时不容许Add
	if w != 0 && delta > 0 && v == int32(delta) {
		panic("sync: WaitGroup misuse: Add called concurrently with Wait")
	}
    // 正常情况下，Add会让v增加，Done会让v减少，如果没有全部Done掉，此处v总是会大于0，直到v为0才往下走
    // w代表有多少个goroutine在等待done信号，wait中通过compareAndSwap对这个w进行加1
	if v > 0 || w == 0 {
		return
	}
	// This goroutine has set counter to 0 when waiters > 0.
	// Now there can't be concurrent mutations of state:
	// - Adds must not happen concurrently with Wait,
	// - Wait does not increment waiters if it sees counter == 0.
	// Still do a cheap sanity check to detect WaitGroup misuse.
    // 当v为0（Done掉了所有）或者w不为0（已经开始等待）才会到这里，但在这个过程中又有一次Add，导致statep变化，所以会panic
	if *statep != state {
		panic("sync: WaitGroup misuse: Add called concurrently with Wait")
	}
	// Reset waiters count to 0.
    // 如果计数值v为0,并且waiter的数量w不为0，那么state的值就是waiter的 数量
    // 将statep重置为0，在Wait中通过这个值来保护信号量发出后还对这个WaitGroup进行操作
	*statep = 0
    // 将信号量发出，触发Wait结束
	for ; w != 0; w-- {
		runtime_Semrelease(semap, false, 0)
	}
}
```

**Done方法**

```go
// Done方法实际就是计数器减1
func (wg *WaitGroup) Done() {
	wg.Add(-1)
}
```

**Wait方法**

```go
// Wait blocks until the WaitGroup counter is zero.
func (wg *WaitGroup) Wait() {
	statep, semap := wg.state()
	for {
		state := atomic.LoadUint64(statep)
		v := int32(state >> 32) // 当前计数值
		w := uint32(state) // waiter的数量
		if v == 0 {
			// Counter is 0, no need to wait.
            // 如果计数值为0，调用这个方法的goroutine不必等待，继续执行后面的逻辑即可 
			return
		}
		// Increment waiters count.
        // 如果statep和state相等，则增加等待计数，同时进入if等待信号量
        // 此处做CAS，主要是防止多个goroutine里要进行wait操作，每有一个goroutine进行wait,等待计数就加1
        // 如果这里不相等，说明statep,在从读出来到CAS比较的这个时间段内，被别的goroutine改写了
        // 则不进入if,继续走for循环，重新再读一次，这样写避免用锁，更高效。
		if atomic.CompareAndSwapUint64(statep, state, state+1) {
			if race.Enabled && w == 0 {
				// Wait must be synchronized with the first Add.
				// Need to model this is as a write to race with the read in Add.
				// As a consequence, can do the write only for the first waiter,
				// otherwise concurrent Waits will race with each other.
				race.Write(unsafe.Pointer(semap))
			}
            // 阻塞休眠，等待信号量
			runtime_Semacquire(semap)
            // 信号量来了，代表所有Add都已经Done
			if *statep != 0 {
                // 走到这里，说明在所有Add都已经Done后，触发信号量后，又被执行了Add
				panic("sync: WaitGroup is reused before previous Wait has returned")
			}
            // 不再阻塞，返回
			return
		}
	}
}
```

### 使用 WaitGroup 时的常见错误

**常见问题一：计数器设置为负值**

WaitGroup计数器的值必须大于1，如果被设置为负值就会panic。

调用Add的时候传递一个负数，或调用Done方法的次数过多，超过了WaitGroup的计数值。

使用 WaitGroup 的正确姿势是，预先确定好 WaitGroup 的计数值，然后调用相同次数的 Done 完成相应的任务

```go
func main() {
	var wg sync.WaitGroup
	wg.Add(1)
	wg.Done()
	wg.Done()
}
// 运行出现panic: sync: negative WaitGroup counter
```

**常见问题二：不期望的 Add 时机**

使用`WaitGroup`时需要遵循的原则就是，等所有的Add方法调用之后再调用Wait，否则会出现panic或不期望的结果

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func dosomething(millisecs time.Duration, wg *sync.WaitGroup) {
	duration := millisecs * time.Millisecond
	time.Sleep(duration) // sleep 一段时间
	wg.Add(1)
	fmt.Println("duration:", duration)
	wg.Done()
}

/*
我们构造这样一个场景：只有部分的 Add/Done 执行完后，Wait 就返回。
我们看一个例子：启动四个 goroutine，每个 goroutine 内部调用 Add(1) 然后调用 Done()，
主 goroutine 调用 Wait 等待任务完成。
*/
func main() {
	// var wg sync.WaitGroup
	// wg.Add(1)
	// wg.Done()
	// wg.Done()
	// panic: sync: negative WaitGroup counter

	var wg sync.WaitGroup
	go dosomething(100, &wg)
	go dosomething(110, &wg)
	go dosomething(120, &wg)
	go dosomething(130, &wg)

	wg.Wait() // 主goroutine等待完成

}

/*
但是它的错误之处在于，将 WaitGroup.Add 方法的调用放在了子 gorotuine 中。
等主 goorutine 调用 Wait 的时候，因为四个任务 goroutine 一开始都休眠，
所以可能 WaitGroup 的 Add 方法还没有被调用，WaitGroup 的计数还是 0，
所以它并没有等待四个子 goroutine 执行完毕才继续执行，而是立刻执行了下一步。
*/

```

**常见问题三：前一个 Wait 还没结束就重用 WaitGroup**

WaitGroup 是可以重用的，只要 WaitGroup 的计数值恢复到零值的状态，那么它就可以被看作是新创建的 WaitGroup，被重复使用。

```go
import (
	"sync"
	"time"
)
func main() {
	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		time.Sleep(time.Millisecond)
		wg.Done() // 计数器减1
		wg.Add(1) // 计数器加1
	}()
	wg.Wait() // 主goroutine等待 有可能和第7行并发执行
}

// panic: sync: WaitGroup is reused before previous Wait has returned
/*
总结一下：WaitGroup 虽然可以重用，但是是有一个前提的，那就是必须等到上一轮的 Wait 完成之后，才能重用 WaitGroup 执行下一轮的 Add/Wait，如果你在 Wait 还没执行完的时候就调用下一轮 Add 方法，就有可能出现 panic。
*/

```

总结一下：WaitGroup 虽然可以重用，但是是有一个前提的，那就是必须等到上一轮的 Wait 完成之后，才能重用 WaitGroup 执行下一轮的 Add/Wait，如果你在 Wait 还没执行完的时候就调用下一轮 Add 方法，就有可能出现 panic。

**noCopy：辅助 vet 检查**

noCopy 字段的类型是 noCopy，它只是一个辅助的、用来帮助 vet 检查用的类型

### 如何避免错误使用WaitGroup

- 不重用 WaitGroup。新建一个 WaitGroup 不会带来多大的资源开销，重用反而更容易出错。
- 保证所有的 Add 方法调用都在 Wait 之前。
- 不传递负数给 Add 方法，只通过 Done 来给计数值减 1。
- 不做多余的 Done 方法调用，保证 Add 的计数值和 Done 方法调用的数量是一样的。
- 不遗漏 Done 方法的调用，否则会导致 Wait hang 住无法返回。
