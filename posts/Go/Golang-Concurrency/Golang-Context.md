---
title: "Golang标准库的Context"
description: "Golang标准库的Context"
date: 2022-09-15 18:10:00
slug: Golang-Contex
image:
categories:
    - Go
tags: ["Go"]
---

### 关于Context

Go1.7将`Context`引入标准库，用来提供上下文信息，专门用来简化 对于处理单个请求的多个 goroutine 之间与请求域的数据、取消信号、截止时间等相关操作，这些操作可能涉及多个 API 调用。

可以使用`WithCancel`、`WithDeadline`、`WithTimeout`或`WithValue`创建的派生上下文。当一个上下文被取消时，它派生的所有上下文也被取消。

Golang issue 28342（https://github.com/golang/go/issues/28342），用来记录当前`Context`存在的问题：

- Context 包名导致使用的时候重复 ctx context.Context；
- Context.WithValue 可以接受任何类型的值，非类型安全；
- Context 包名容易误导人，实际上，Context 最主要的功能是取消 goroutine 的执行；
- Context 漫天飞，函数污染。

以下场景可以考虑使用Context：

- 上下文信息传递 （request-scoped），比如处理 http 请求、在请求处理链路上传递信息；
- 控制子 goroutine 的运行；
- 超时控制的方法调用；
- 可以取消的方法调用。

### Context接口

`Context`源码在context.Context下，Context是一个接口，具体实现包括 4 个方法，分别是 Deadline、Done、Err 和 Value。

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key any) any
}
```

- Deadline()：方法会返回这个 Context 被取消的截止日期。如果没有设置截止日期，ok 的值是 false。后续每次调用这个对象的 Deadline 方法时，都会返回和第一次调用相同的结果。
- Done()：方法返回一个 Channel 对象。在 Context 被取消时，此 Channel 会被 close，如果没被取消，可能会返回 nil。后续的 Done 调用总是返回相同的结果。当 Done 被 close 的时候，你可以通过 ctx.Err 获取错误信息。
- Err()：返回当前`Context`结束的原因，它只会在`Done`返回的Channel被关闭时才会返回非空的值；如果当前`Context`被取消就会返回`Canceled`错误；如果当前`Context`超时就会返回`DeadlineExceeded`错误；
- Value(key any) any： 返回此 ctx 中和指定的 key 相关联的 value。

###context.TODO()和context.Background()

两个常用的生成顶层Context的方法：

- context.TODO()：返回一个非 nil 的、空的 Context，没有任何值，不会被 cancel，不会超时，没有截止日期。当你不清楚是否该用 Context，或者目前还不知道要传递一些什么上下文信息的时候，就可以使用这个方法。
-  context.Background()：返回一个非 nil 的、空的 Context，没有任何值，不会被 cancel，不会超时，没有截止日期。一般用在主函数、初始化、测试以及创建根 Context 的时候。

`background`和`todo`本质上都是`emptyCtx`结构体类型，是一个不可取消，没有设置截止时间，没有携带任何值的Context。

两个的底层实现是一模一样的。

```go
var (
	background = new(emptyCtx)
	todo       = new(emptyCtx)
)
func Background() Context {
	return background
}
func TODO() Context {
	return todo
}
```

### WithValue 函数

基于`parent Context`生成了一个新的Context，常用于保存键值对，key-value。

链式查找：覆盖了Value方法，优先从自己的key中查找，不存在再从parent中查找。

```go
func WithValue(parent Context, key, val any) Context {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	if key == nil {
		panic("nil key")
	}
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}
type valueCtx struct {
	Context
	key, val any
}
```

#### 示例

```go
// 在上下文中使用数据
func doSomething(ctx context.Context) {
	fmt.Println("Doing something: myKey's value is ", ctx.Value("myKey"))
	// 链式查找
	anotherCtx := context.WithValue(ctx, "myKey", "antherValue")
	doAnother(anotherCtx)
	fmt.Println("Doing something: myKey's value is ", ctx.Value("myKey"))
}
func doAnother(ctx context.Context) {
	fmt.Println("Doing another myKey is ", ctx.Value("myKey"))
}

func main() {
	// Context 中实现了 2 个常用的生成顶层 Context 的方法。context.TODO()和context.Background()
	// ctx := context.TODO()
	ctx := context.Background()
	/*
	WithValue 基于 parent Context 生成一个新的 Context，保存了一个 key-value 键值对。它常常用来传递上下文。
	*/
	ctx = context.WithValue(ctx, "myKey", "myValue")
	doSomething(ctx)
}
```

### WithCancel函数

WithCancel 方法返回 parent 的副本，只是副本中的 Done Channel 是新建的对象，它的类型是 cancelCtx。

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	c := newCancelCtx(parent)
	propagateCancel(parent, &c)
	return &c, func() { c.cancel(true, Canceled) }
}
// newCancelCtx returns an initialized cancelCtx.
func newCancelCtx(parent Context) cancelCtx {
	return cancelCtx{Context: parent}
}

```

`propagateCancel` 方法会顺着 parent 路径往上找，直到找到一个 cancelCtx，或者为 nil。如果不为空，就把自己加入到这个 cancelCtx 的 child，以便这个 cancelCtx 被取消的时候通知自己。如果为空，会新起一个 goroutine，由它来监听 parent 的 Done 是否已关闭。

cancelCtx 是向下传递的，如果一个WithCancel生成的Context被cancel，如果它的子Context也是cancelCtx类型也会被cancel掉，parent Context 不会因为子 Context 被 cancel 而 cancel。

注意：只要任务完成了就需要调用`Cancel`，切记一定要尽早释放。

#### 示例

```go
// WithCancel 方法返回 parent 的副本，只是副本中的 Done Channel 是新建的对象，它的类型是 cancelCtx。
func doSomething(ctx context.Context) {
	ctx, cancel := context.WithCancel(ctx)
	printCh := make(chan int)
	go doAnother(ctx, printCh)
	for num := 1; num <= 3; num++ {
		printCh <- num
	}
	cancel()
	time.Sleep(100 * time.Millisecond)
	fmt.Println("doSomething finished!")
}
// 方法返回一个 Channel 对象。在 Context 被取消时，此 Channel 会被 close，如果没被取消，可能会返回 nil。
// 后续的 Done 调用总是返回相同的结果。当 Done 被 close 的时候，你可以通过 ctx.Err 获取错误信息。
func doAnother(ctx context.Context, printCh <-chan int) {
	for {
		select {
		case <-ctx.Done():
			if err := ctx.Err(); err != nil {
				fmt.Printf("doAnother err:%s\n", err)
			}
			fmt.Println("doAnther finished!")
			return
		case num := <-printCh:
			fmt.Println("doAnother:", num)
		}
	}
}
// 结束上下文
func main() {
	ctx := context.Background()
	doSomething(ctx)
}
/*
doAnother: 1
doAnother: 2
doAnother: 3
doAnother err:context canceled
doAnther finished!
doSomething finished!
*/
```

### WithDeadline和WithTimeout函数

- context.WithDeadline 给上下文一个截止日期
- context.WithTimeout 给上下文一个时间限制

WithDeadline 会返回一个 parent 的副本，并且设置了一个不晚于参数 d 的截止时间，类型为 timerCtx（或者是 cancelCtx）。（注意：如果它的截止时间晚于parent的截止时间，则以parent的为准）

如果当前时间已经超过了截止时间，就直接返回一个已经被 cancel 的 timerCtx。否则就会启动一个定时器，到截止时间取消这个 timerCtx。

WithTimeout和WithDeadline实现上是一样的，

```go
// WithTimeout 实现 
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}
```

```go
// WithDeadline实现
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
    // 如果parent的截止时间早于它的，则直接返回cancelCtx
	if cur, ok := parent.Deadline(); ok && cur.Before(d) {
		// The current deadline is already sooner than the new one.
		return WithCancel(parent)
	}
	c := &timerCtx{
		cancelCtx: newCancelCtx(parent),
		deadline:  d,
	}
    // 同cancelCtx的处理逻辑：
    //propagateCancel 方法会顺着 parent 路径往上找，直到找到一个 cancelCtx，或者为 nil。如果不为空，就把自己加入到这个 cancelCtx 的 child，以便这个 cancelCtx 被取消的时候通知自己。如果为空，会新起一个 goroutine，由它来监听 parent 的 Done 是否已关闭。
	propagateCancel(parent, c)
	dur := time.Until(d)
    // 当前时间已经超过截止时间直接cancel
	if dur <= 0 {
		c.cancel(true, DeadlineExceeded) // deadline has already passed
		return c, func() { c.cancel(false, Canceled) }
	}
	c.mu.Lock()
	defer c.mu.Unlock()
	if c.err == nil {
        // 设定一个计时器，到截止时间直接返回
		c.timer = time.AfterFunc(dur, func() {
			c.cancel(true, DeadlineExceeded)
		})
	}
	return c, func() { c.cancel(true, Canceled) }
}
```

timerCtx 的 Done 被 Close 掉，主要是由下面的某个事件触发的：

- 截止时间到了
- cancel函数被调用
- parent的Done被close

#### 示例

```go
// context.WithDeadline 给上下文一个截止日期
// 如果当前时间已经超过了截止时间，就直接返回一个已经被 cancel 的 timerCtx。
// 否则就会启动一个定时器，到截止时间取消这个 timerCtx。

// context.WithTimeout 给上下文一个时间限制
func doSomething(ctx context.Context) {
	// deadline := time.Now().Add(1500 * time.Millisecond)
	// ctx, cancel := context.WithDeadline(ctx, deadline)
	ctx, cancel := context.WithTimeout(ctx, 1500*time.Millisecond)
	// defer cancel()
	printCh := make(chan int)
	go doAnother(ctx, printCh)
	for num := 1; num <= 3; num++ {
		select {
		case printCh <- num:
			time.Sleep(1 * time.Second)
		case <-ctx.Done():
			break
		}
	}
	cancel()
	time.Sleep(100 * time.Millisecond)
	fmt.Println("doSomething finished!")
}
func doAnother(ctx context.Context, printCh <-chan int) {
	for {
		select {
		case <-ctx.Done():
			if err := ctx.Err(); err != nil {
				fmt.Printf("doAnother err:%s\n", err)
			}
			fmt.Println("doAnther finished!")
			return
		case num := <-printCh:
			fmt.Println("doAnother:", num)
		}
	}
}
func main() {
	ctx := context.Background()
	doSomething(ctx)
}
/*
doAnother: 1
doAnother: 2
doAnother err:context deadline exceeded
doAnther finished!
doSomething finished!
*/
```

**切记：WithDeadline和WithTimeout返回的cancel一定要调用**

### 使用`Context`的一些注意事项

- 使用Context时会把它放在第一个参数的位置
- 从来不把 nil 当做 Context 类型的参数值，可以使用 context.Background() 创建一个空的上下文对象，也不要使用 nil。
- Context 只用来临时做函数之间的上下文透传，不能持久化 Context 或者把 Context 长久保存。把 Context 持久化到数据库、本地文件或者全局变量、缓存中都是错误的用法。
- key 的类型不应该是字符串类型或者其它内建类型，否则容易在包之间使用 Context 时候产生冲突。使用 WithValue 时，key 的类型应该是自己定义的类型。
- 常常使用 struct{}作为底层类型定义 key 的类型。对于 exported key 的静态类型，常常是接口或者指针。这样可以尽量减少内存分配。
- Context是线程安全的，可以放心的在多个goroutine中传递