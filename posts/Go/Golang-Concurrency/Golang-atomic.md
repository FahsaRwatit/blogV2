---
title: "Golang的原子操作atomic"
description:  "Golang的原子操作atomic"
date: 2022-09-18 18:10:00
slug: Golang-atomic
image:
categories:
    - Go
tags: ["Golang"]

---

[未完成...]

Go语言中的原子操作由标准库内置的`sync/atomic/`包提供。

通常使用原子操作比使用锁的效率更高。

**atomic 操作的对象是一个地址，你需要把可寻址的变量的地址作为参数传递给方法，而不是把变量的值传递给方法**

### Add系列方法





### CAS （CompareAndSwap）系列方法





### Swap系列方法



### Load系列方法











### Store系列方法







### Value类型

atomic 还提供了一个特殊的类型：Value。它可以原子地存取对象类型，但也只能存取，不能 CAS 和 Swap，常常用在配置变更等场景中。

```go
type Value struct {
	v any
}
func (v *Value) Load() (val any) {}
func (v *Value) Store(val any) {}
```

#### 示例

```go
type Config struct {
	NodeName string
	Addr     string
	Count    int32
}
func LoadNewConfig() Config {
	return Config{
		NodeName: "北京",
		Addr:     "10.23.25.108",
		Count:    rand.Int31(),
	}
}
// atomic.Value的Load/Store的使用
func main() {
	var config atomic.Value
	config.Store(LoadNewConfig())
	var cond = sync.NewCond(&sync.Mutex{})
	// 设置新的config
	go func() {
		for {
			time.Sleep(time.Duration(5+rand.Int63n(5)) * time.Second)
			config.Store(LoadNewConfig())
			// 通知等待者条件已变更
			cond.Broadcast()
		}
	}()
	go func() {
		cond.L.Lock()
		cond.Wait()
		c := config.Load().(Config)
		fmt.Printf("new config:%v\n", c) // new config:{北京 10.23.25.108 1427131847}
		cond.L.Unlock()
	}()
	select {}
}
```







