---
title: "Golang中的Channel"
description: "Golang中的Channel"
date: 2022-08-18 18:10:00
slug: golang-channel
image:
categories:
    - Go
tags: ["Go"]
---

CSP 是 Communicating Sequential Process 的简称，中文直译为通信顺序进程，或者叫做交换信息的循序进程，是用来描述并发系统中进行交互的一种模式。

最早出现在计算机科学家 Tony Hoare 在 1978 年发表的论文中（https://www.cs.cmu.edu/~crary/819-f09/Hoare78.pdf）

**CSP 允许使用进程组件来描述系统，它们独立运行，并且只通过消息传递的方式通信。**

### 关于Channel

Don’t communicate by sharing memory, share memory by communicating.

执行业务处理的 goroutine 不要通过共享内存的方式通信，而是要通过 Channel 通信的方式分享数据。

**不要通过共享内存的方式进行通信，而是应该通过通信的方式共享内存**

1、示例：通过共享内存的方式进行通信

```go
func watch(p *int) {
	for true {
		if *p == 1 {
			fmt.Println("hello")
			break
		}
	}
}
func main() {
	i := 0
	go watch(&i)
	time.Sleep(1 * time.Second)
	i = 1
	time.Sleep(1 * time.Second)
}
```

2、示例：通过Channel通信的方式共享内存

```go
// 通过Channel通信的方式共享内存
func watch(ch chan int) {
	if <-ch == 1 {
		fmt.Println("hello")
	}
}
func main() {
	var ch = make(chan int)
	go watch(ch)
	time.Sleep(1 * time.Second)
	ch <- 1
	time.Sleep(1 * time.Second)
}
```

使用通信来共享内存的优势：

- 避免协程竞争和数据冲突的问题
- 更高级的抽象，降低开发难度，增加程序可读性
- 模块之间更容易解耦，增强扩展性和可维护性

### Channel基本用法

#### 示例

```go
func main() {
	// 声明方式
	// ChanType = (chan | chan<- | <-chan) ElementType
	// var Channel_name ChanType

	chan1 := make(chan int)
	chan2 := make(chan<- int)
	chan3 := make(<-chan int)
	fmt.Println(chan1, chan2, chan3)
	// 这个箭头总是射向左边的，元素类型总在最右边。如果箭头指向 chan，就表示可以往 chan 中塞数据；如果箭头远离 chan，就表示 chan 会往外吐数据
	// 有缓冲管道chan
	chan4 := make(chan struct{}, 1)
	// cap：返回chan容量，len:返回chan长度
	fmt.Println(len(chan4)) // 0
	fmt.Println(cap(chan4)) // 1

	chan4 <- struct{}{} // 往管道中发送数据

	fmt.Println(len(chan4)) // 1
	fmt.Println(cap(chan4)) // 1

	fmt.Println(<-chan4) // 从管道中读数据 {}

	fmt.Println(len(chan4)) // 0
	fmt.Println(cap(chan4)) // 1

	// select操作
	fmt.Println()
	var ch = make(chan int, 10)
	for i := 0; i < 10; i++ {
		select {
		case ch <- i:
		case v := <-ch:
			fmt.Println(v)
		}
	}
	fmt.Println()
	var ch1 = make(chan int, 3)
	ch1 <- 11
	ch1 <- 22
	ch1 <- 33
	// // for-range
	// for v := range ch1 {
	// 	fmt.Println(v)
	// }
	// // 清空chan
	// for range ch1 {
	// }
	v, ok := <-ch1
	fmt.Println(v, ok) // 11 true
	close(ch1)
	// 关闭chan后，缓冲区有值依然能够读取，且ok=true
	v, ok = <-ch1
	fmt.Println(v, ok) // 22 true
	v, ok = <-ch1
	fmt.Println(v, ok) // 33 true

	// 关闭chan后，缓冲区无值返回对应类型的默认值，且ok=false
	v, ok = <-ch1
	fmt.Println(v, ok) // 0 false

}
```

`Channel`可以声明三种类型：可以接收和 发送、只能发送、只能接收

```go
ChanType = (chan | chan<- | <-chan) ElementType
```

这个箭头总是射向左边的，元素类型总在最右边。如果箭头指向 chan，就表示可以往 chan 中塞数据；如果箭头远离 chan，就表示 chan 会往外吐数据。

```go
ch1 <- // 发送数据
<- ch1 // 接收数据,丢弃接收的一条数据
s := ch1 <- struct{}{} // 接收数据并赋值给变量s
```

Go 内建的函数 close、cap、len 都可以操作 chan 类型：close 会把 chan 关闭掉，cap 返回 chan 的容量，len 返回 chan 中缓存的还未被取走的元素数量。

```go
fmt.Println(len(chan4))
fmt.Println(cap(chan4))
close(ch1)
v, ok = <-ch1 // 返回值为2个
```



### Channel实现原理

`Channel是Go的一等公民`，Channel的设计：Ring Buffer、Receive Queue、Send Queue、Mutex、closed。

环形缓存可以大幅降低FGC的开销。

互斥锁：

- 互斥锁并不是排队发送/接收数据
- 互斥锁保护的hchan结构体本身
- Channel并不是无锁的

数据类型为：`runtime/hchan`，源码位于`runtime/chan.go`

### hchan

```go
type hchan struct {
	qcount   uint // 循环队列元素的数量
	dataqsiz uint // 循环队列的大小
	buf      unsafe.Pointer // 循环队列buf的指针
	elemsize uint16 // chan中元素的大小
	closed   uint32 // 是否已经关闭，1 关闭 0 未关闭
	elemtype *_type // chan中元素类型
	sendx    uint   // sned在buf中的索引
	recvx    uint   // recv在buf中的索引
	recvq    waitq  // receiver的等待队列
	sendq    waitq  // sender的等待队列
    
	lock mutex // 互斥锁保护所有字段
}
```

- `qcount`：代表chan中已经接收但还 未被取走的元素，函数`len()`可返回这个字段的值。
- `dataqsiz`：队列的大小。chan 使用一个循环队列来存放元素，循环队列很适合这种生产者 - 消费者的场景。
- `buf`：存放元素循环队列的buffer
- elemtype 和 elemsize：chan 中元素的类型和 大小。 chan 一旦声明，它的元素类型是固定的，即普通类型或者指针类型，所以元素大小也是固定的。
- sendx：处理发送数据的指针在 buf 中的位置。一旦接收了新的数据，指针就会加上 elemsize，移向下一个位置。buf 的总大小是 elemsize 的整数倍，而且 buf 是一个循环列表。
- recvx：处理接收请求时的指针在 buf 中的位置。一旦取出数据，此指针会移动到下一个位置。
- recvq：chan 是多生产者多消费者的模式，如果消费者因为没有数据可读而被阻塞了，就会被加入到 recvq 队列中。
- sendq：如果生产者因为 buf 满了而阻塞，会被加入到 sendq 队列中。

### makechan 

只需要关注 makechan 就好了，因为 makechan64 只是做了 size 检查，底层还是调用 makechan 实现的。

`makechan`主要是生成`hchan`对象。会根据chan的容量大小和元素类型不同，初始化不同的存储空间。

```go
func makechan(t *chantype, size int) *hchan {
	elem := t.elem
    // 编译器检查代码
	// compiler checks this but be safe.
	if elem.size >= 1<<16 {
		throw("makechan: invalid channel element type")
	}
	if hchanSize%maxAlign != 0 || elem.align > maxAlign {
		throw("makechan: bad alignment")
	}

	mem, overflow := math.MulUintptr(elem.size, uintptr(size))
	if overflow || mem > maxAlloc-hchanSize || size < 0 {
		panic(plainError("makechan: size out of range"))
	}

	// Hchan does not contain pointers interesting for GC when elements stored in buf do not contain pointers.
	// buf points into the same allocation, elemtype is persistent.
	// SudoG's are referenced from their owning thread so they can't be collected.
	// TODO(dvyukov,rlh): Rethink when collector can move allocated objects.
	var c *hchan
	switch {
	case mem == 0:
		// Queue or element size is zero.
        // chan的size或元素的size是0，不必创建 buf
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		// Race detector uses this location for synchronization.
		c.buf = c.raceaddr()
	case elem.ptrdata == 0:
		// Elements do not contain pointers.
		// Allocate hchan and buf in one call.
        // 元素不是指针，分配一块连续的内存给hchan和buf
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
        // hchan数据结构后面紧跟着就是buf
		c.buf = add(unsafe.Pointer(c), hchanSize)
	default:
		// Elements contain pointers.
        // 元素包含指针就单独分配buf
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}
	// 元素大小、类型、容量都记录下来
	c.elemsize = uint16(elem.size)
	c.elemtype = elem
	c.dataqsiz = uint(size)
	lockInit(&c.lock, lockRankHchan)

	if debugChan {
		print("makechan: chan=", c, "; elemsize=", elem.size, "; dataqsiz=", size, "\n")
	}
	return c
}
```

### send

向chan中发送数据的时候，会执行`chansend1`函数，chansend1 会调用`chansend`

`c<-`关键字是一个语法糖，编译阶段，会把c<-转化为runtime.chansend1()，chansend1()会调用chansend()方法。

`Channel`发送分为3种情况、1、直接发送   2、放入缓存  3、休眠等待。

#### 直接发送

实现原理：发送数据前，已经有G在休眠等待接收，此时缓存肯定是空的，不用考虑缓存，将数据直接拷贝给G的接收变量，唤醒G。

实现：从队列里取出一个等待接收的G，将数据直接拷贝到接收变量中，唤醒G。

#### 放入缓存

原理：没有G在休眠等待，但是有缓存空间，将数据放入缓存。

实现：获取可存入的缓存地址、存入数据、维护索引

#### 休眠等待

原理：没有G在休眠等待，而且没有缓存或满了，自己进入发送队列休眠等待

实现：把自己包装成sudog，将sudog放入sendq队列，休眠并解锁，（什么时候解锁，被唤醒后数据已经被取走，维护其他数据）

```go
//go:nosplit
func chansend1(c *hchan, elem unsafe.Pointer) {
	chansend(c, elem, true, getcallerpc())
}
```

去除掉一些调试代码后的chansend：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // 第一部分 
    // 如果chan为nil就把调用者(goroutine)阻塞休眠(park)，调用者就永远被阻塞了
	if c == nil {
		if !block {
			return false
		}
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
        // 被阻塞了，不可能执行
		throw("unreachable") 
	}
    // 第二部分 
    // 如果chan没有被close并且chan满了，直接返回。
    // 当你往一个已经满了的 chan 实例发送数据时，并且想不阻塞当前调用，那么这里的逻辑是直接返回。
    // chansend1 方法在调用 chansend 的时候设置了阻塞参数block=true，所以不会执行到第二部分的分支里。
	if !block && c.closed == 0 && full(c) {
		return false
	}

	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}
     // 第三部分 
    // chan已经被close的情景，再发数据会panic
	lock(&c.lock)
	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("send on closed channel"))
	}
    // 第四部分 
	// 从接收队列中出队一个等待的receiver
    // 如果等待队列中有等待的 receiver，那么这段代码就把它从队列中弹出，然后直接把数据交给它（通过 memmove(dst, src, t.size)），而不需要放入到 buf 中，速度可以更快一些。
	if sg := c.recvq.dequeue(); sg != nil {
		// Found a waiting receiver. We pass the value we want to send
		// directly to the receiver, bypassing the channel buffer (if any).
		send(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true
	}
	// 第五部分，buf未满
    // 当前没有receiver，且buf未满，则把数据放入buf中，返回成功
	if c.qcount < c.dataqsiz {
		// Space is available in the channel buffer. Enqueue the element to send.
		qp := chanbuf(c, c.sendx)
		if raceenabled {
			racenotify(c, c.sendx, nil)
		}
		typedmemmove(c.elemtype, qp, ep)
		c.sendx++
		if c.sendx == c.dataqsiz {
			c.sendx = 0
		}
		c.qcount++
		unlock(&c.lock)
		return true
	}
    // 第六部分是处理 buf 满的情况。
	// chansend1不会进入这里 ，因为block=true
    // 如果 buf 满了，发送者的 goroutine 就会加入到发送者的等待队列中，直到被唤醒。
    // 这个时候，数据或者被取走了，或者 chan 被 close 了。
	if !block {
		unlock(&c.lock)
		return false
	}
    ......
}
```

### recv

在处理从 chan 中接收数据时，Go 会把代码转换成 chanrecv1 函数，如果要返回两个返回值，会转换成 chanrecv2，chanrecv1 函数和 chanrecv2 会调用 chanrecv。

`<-c`关键字是一个语法糖

编译阶段，i <- c 转化为runtime.chanrecv1()

编译阶段，i，ok <-c转化为runtime.chanrecv2()

最后会调用chanrecv()方法。

`Channel`接收分为4种情况：

- 有等待的G，从G接收
- 有等待的G，从缓存接收
- 接收缓存
- 阻塞接收

#### 有等待的G，从G接收

原理：

- 接收数据前，已经有G在休眠等待发送
- 而且这个Channel没有缓存
- 将数据直接从G拷贝过来，唤醒G

实现：

- 判断有G在发送队列等待，进入recv()
- 判断此Channel无缓存
- 直接从等待的G取走数据，唤醒G

#### 有等待的G，从缓存接收

原理：

- 接收数据前，已经有G在休眠等待发送
- 而且这个Channel有缓存
- 从缓存取走一个数据
- 将休眠的G的数据放进缓存，唤醒G

实现：

- 判断有G在发送队列等待，进入recv()
- 判断此Channel有缓存
- 从缓存中取走一个数据
- 将G的数据放入缓存，唤醒G

#### 接收缓存

原理：

- 没有G在休眠等待发送，但是缓存有内容
- 从缓存取走数据

实现：

- 判断没有G在发送队列等待
- 判断此Channel有缓存
- 从缓存中取走一个数据

#### 阻塞接收

原理：

- 没有G在休眠等待，而且没有缓存或缓存为空
- 自己进入接收队列，休眠等待

 实现：

- 判断没有G在发送队列等待
- 判断此Channel无缓存
- 将自己包装成sudog
- 将sudog放入接收等待队列，休眠。（唤醒时，发送的G已经把数据拷贝到位）

```go
//go:nosplit
func chanrecv1(c *hchan, elem unsafe.Pointer) {
	chanrecv(c, elem, true)
}

//go:nosplit
func chanrecv2(c *hchan, elem unsafe.Pointer) (received bool) {
	_, received = chanrecv(c, elem, true)
	return
}
```

#### chanrecv

chanrecv1 和 chanrecv2 传入的 block 参数的值是 true，都是阻塞方式，所以我们分析 chanrecv 的实现的时候，不考虑 block=false 的情况。

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
    // 第一部分，chan为nil
    // 和send一样，从nil chan中接收（读取、获取）数据时，调用者会被永久阻塞！
	if c == nil {
		if !block {
			return
		}
		gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

    // 第二部分，block=false,且c为空
    // 因为block=false所以这部分直接忽略
	// Fast path: check for failed non-blocking operation without acquiring the lock.
	if !block && empty(c) {
		if atomic.Load(&c.closed) == 0 {
			return
		}
		if empty(c) {
			// The channel is irreversibly closed and empty.
			if raceenabled {
				raceacquire(c.raceaddr())
			}
			if ep != nil {
				typedmemclr(c.elemtype, ep)
			}
			return true, false
		}
	}

	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}
	// 加锁，返回时释放锁 
	lock(&c.lock)
	if c.closed != 0 {
         // 第三部分，chan已经被close,且chan为空
		if c.qcount == 0 {
			if raceenabled {
				raceacquire(c.raceaddr())
			}
			unlock(&c.lock)
			if ep != nil {
				typedmemclr(c.elemtype, ep)
			}
			return true, false
		}
		// The channel has been closed, but the channel's buffer have data.
	} else {
		// Just found waiting sender with not closed.
        // 第四部分,是处理 sendq 队列中有等待者的情况。
        // 如果缓冲区大小为 0，则直接从sendq等待队列中弹出一个sender给receiver。
        // 否则，直接从buf中读取数据
        // 从队列头部接收并将发送者的值添加到队列尾部（两者都映射到相同的缓冲槽，因为队列已满）。
		if sg := c.sendq.dequeue(); sg != nil {
			// Found a waiting sender. If buffer is size 0, receive value
			// directly from sender. Otherwise, receive from head of queue
			// and add sender's value to the tail of the queue (both map to
			// the same buffer slot because the queue is full).
			recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
			return true, true
		}
	}
	// 第五部分，没有等待的sender，buf中有数据
    // 这个是和 chansend 共用一把大锁，所以不会有并发的问题。如果 buf 有元素，就取出一个元素给 receiver。
	if c.qcount > 0 {
        // 直接从队列中接收 
		// Receive directly from queue
		qp := chanbuf(c, c.recvx)
		if raceenabled {
			racenotify(c, c.recvx, nil)
		}
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
		typedmemclr(c.elemtype, qp)
		c.recvx++
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		c.qcount--
		unlock(&c.lock)
		return true, true
	}

	if !block {
		unlock(&c.lock)
		return false, false
	}

    // 第六部分，buf中没有数据，且没有sender时，receiver阻塞,直到它从sender中接收到了数据或者chan被close，才返回。
	// no sender available: block on this channel.
	......
}
```

### close

关闭chan时运行`closechan`函数。

去除掉调试代码后：

```go
func closechan(c *hchan) {
    // 如果chan为nil,则panic
	if c == nil {
		panic(plainError("close of nil channel"))
	}
	// chan已经close,则panic
	lock(&c.lock)
	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("close of closed channel"))
	}

	c.closed = 1

	var glist gList

	// release all readers
    // 释放所有的readers
	for {
		sg := c.recvq.dequeue()
		if sg == nil {
			break
		}
		if sg.elem != nil {
			typedmemclr(c.elemtype, sg.elem)
			sg.elem = nil
		}
		if sg.releasetime != 0 {
			sg.releasetime = cputicks()
		}
		gp := sg.g
		gp.param = unsafe.Pointer(sg)
		sg.success = false
		if raceenabled {
			raceacquireg(gp, c.raceaddr())
		}
		glist.push(gp)
	}

	// release all writers (they will panic)
    // 释放所有的writers，它们会panic
	for {
		sg := c.sendq.dequeue()
		if sg == nil {
			break
		}
		sg.elem = nil
		if sg.releasetime != 0 {
			sg.releasetime = cputicks()
		}
		gp := sg.g
		gp.param = unsafe.Pointer(sg)
		sg.success = false
		if raceenabled {
			raceacquireg(gp, c.raceaddr())
		}
		glist.push(gp)
	}
	unlock(&c.lock)

	// Ready all Gs now that we've dropped the channel lock.
	for !glist.empty() {
		gp := glist.pop()
		gp.schedlink = 0
		goready(gp, 3)
	}
}
```

close chan 的主要逻辑。如果 chan 为 nil，close 会 panic；如果 chan 已经 closed，再次 close 也会 panic。否则的话，如果 chan 不为 nil，chan 也没有 closed，就把等待队列中的 sender（writer）和 receiver（reader）从队列中全部移除并唤醒。

###Channel中容易犯的错误

**使用 Channel 最常见的错误是 panic 和 goroutine 泄漏**

`panic`的情况有三种：

- close 为 nil 的 chan；
- send 已经 close 的 chan；
- close 已经 close 的 chan。

#### goroutine泄露问题

示例：

```go
// goroutine 泄露问题 示例
// 因为chan为无缓冲channel,如果超时chan从来没有被读取。
// 如果业务处理完往chan中发送通知，将造成永久阻塞，进而导致goroutine泄露
// 解决这个bug的方法就是将unbuffered chan改为容量为1的chan
func process() bool {
	ch := make(chan bool)
	timeout := 3 * time.Second
	go func() {
		// 模拟处理耗时的业务
		time.Sleep(timeout + time.Second)
		ch <- true
		fmt.Println("exit goroutine")

	}()
	select {
	case v := <-ch:
		return v
	case <-time.After(timeout):
		return false
	}
}
func main() {
	res := process()
	fmt.Println(res) // false
}
```

### 选择使用Channel的方法

- 共享资源的并发访问使用传统并发原语；
- 复杂的任务编排和消息传递使用 Channel；
- 消息通知机制使用 Channel，除非只想 signal 一个 goroutine，才使用 Cond；
- 简单等待所有任务的完成用 WaitGroup，也有 Channel 的推崇者用 Channel，都可以；
- 需要和 Select 语句结合，使用 Channel；
- 需要和超时配合时，使用 Channel 和 Context。
