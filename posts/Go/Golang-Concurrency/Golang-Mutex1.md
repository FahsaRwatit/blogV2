---
title: "Golang中使用Mutex"
description: "Golang中使用Mutex"
date: 2022-09-10 18:10:00
slug: Golang-Mutex1
image:
categories:
    - Go
tags: ["Go","Mutex"]

---

### Go并发场景中不使用锁的例子

- count++ 不是一个原子操作
- 运行go run -race mutex1_1.go 会输出警告信息，告诉你会有并发问题
- race这个工具只有实际对地址进行访问的时候才能检测到，并不能在编译的时候发现data race问题，并且部署在线上比较影响性能
- 运行 go tool compile -race -S mutex1_1.go 可以查看计数器例子编译后的代码，重点关注count++前后的代码

```go
import (
	"fmt"
	"sync"
)

// 并发场景中不使用锁的例子
// count++ 不是一个原子操作
// 运行go run -race mutex1_1.go 会输出警告信息，告诉你会有并发问题
// race这个工具只有实际对地址进行访问的时候才能检测到，并不能在编译的时候发现data race问题，并且部署在线上比较影响性能
// 运行 go tool compile -race -S mutex1_1.go 可以查看计数器例子编译后的代码，重点关注count++前后的代码
func main() {
	var count = 0
	// 使用waitGroup等待10个groutine完成
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for i := 0; i < 100000; i++ {
				count++
			}
		}()
	}
	wg.Wait()
	fmt.Println(count)
	fmt.Println("End...")
}
```

### 使用mutex解决data race的问题

- 运行go run -race mutex1_2.go发现没有警告信息了
- 注意：Mutex的零值是还没有goroutine等待的未加锁的状态，所以不需要额外的初始化，直接声明就可以:var mu sync.Mutex

```go
import (
	"fmt"
	"sync"
)

// 使用mutex解决data race的问题
// 运行go run -race mutex1_2.go发现没有警告信息了
// 注意：Mutex的零值是还没有goroutine等待的未加锁的状态，所以不需要额外的初始化，直接声明就可以:var mu sync.Mutex
func main() {
	// 互斥锁保护计数器
	var mu sync.Mutex
	var count = 0
	// 使用waitGroup等待10个groutine完成
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for i := 0; i < 100000; i++ {
				// 加锁
				mu.Lock()
				count++
				// 解锁
				mu.Unlock()
			}
		}()
	}
	wg.Wait()
	fmt.Println(count)
	fmt.Println("End...")
}
```

### Mutex嵌入到其他struct中使用的例子

-  采用嵌入字段的方式，可以在这个struct上直接调用Lock()/Unlock()
- 如果嵌入字段有多个，一般会将Mutex放在要控制的字段上面然后使用空格分隔开来

```go
import (
	"fmt"
	"sync"
)
// Mutex嵌入到其他struct中使用的例子
// 采用嵌入字段的方式，可以在这个struct上直接调用Lock()/Unlock()
// 如果嵌入字段有多个，一般会将Mutex放在要控制的字段上面然后使用空格分隔开来
type Counter struct {
	sync.Mutex
	Count uint64
}

func main() {
	var counter Counter
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for i := 0; i < 100000; i++ {
				// 加锁
				counter.Lock()
				counter.Count++
				// 解锁
				counter.Unlock()
			}
		}()
	}
	wg.Wait()
	fmt.Println(counter.Count)
	fmt.Println("End...")
}
```

### 实现线程安全的计数器类型

- 把获取锁，释放锁 ，计数加1的方法封装起来，对外不需要暴露锁等逻辑
- 推荐文章： sync.Mutex源代码分析：https://colobu.com/2018/12/18/dive-into-sync-mutex/

```go
import (
	"fmt"
	"sync"
)

// 把获取锁，释放锁 ，计数加1的方法封装起来，对外不需要暴露锁等逻辑
// 推荐文章： sync.Mutex源代码分析：https://colobu.com/2018/12/18/dive-into-sync-mutex/

// 线程安全的计数器类型
type Counter struct {
	CounterType int
	Name        string

	mu    sync.Mutex
	count int64
}

// 加1的方法，内部使用互斥锁保护
func (c *Counter) Incr() {
	c.mu.Lock()
	c.count++
	c.mu.Unlock()
}

// 得到计数器的值，也需要锁保护
func (c *Counter) Count() int64 {
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.count

}

func main() {
	var counter Counter
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for i := 0; i < 100000; i++ {
				counter.Incr()
			}
		}()
	}
	wg.Wait()
	fmt.Println(counter.Count())
	fmt.Println("End...")
}
```
