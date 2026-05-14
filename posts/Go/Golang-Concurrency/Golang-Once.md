---
title: "Go标准库中的Once"
description: "Go标准库中的Once"
date: 2022-09-17 18:10:00
slug: Golang-Once
image:
categories:
    - Go
tags: ["Go"]
---

### 单例初始化示例

```go
// 单例资源初始化的常见示例
// 使用互斥锁保证线程(goroutine)的安全
var connMu sync.Mutex
var conn net.Conn

func getConn() net.Conn {
	connMu.Lock()
	defer connMu.Unlock()
	// 返回已创建好的连接
	if conn != nil {
		return conn
	}
	conn, _ := net.DialTimeout("tcp", "baidu.com:80", 10*time.Second)
	return conn
}

func main() {
	// 使用连接
	conn = getConn()
	if conn == nil {
		panic("conn is nil")
	}
}
```

### Once的使用

```go
func main() {
	var once sync.Once
	// 第一个初始化函数
	f1 := func() {
		fmt.Println("f1")
	}
	once.Do(f1)
	// 第二个初始化函数
	f2 := func() {
		fmt.Println("f2")
	}
	// 第二个初始化函数， 无输出
	once.Do(f2)

	// 通过闭包方式使用
	var addr = "baidu.com:80"
	var conn net.Conn
	var err error
	var once1 sync.Once
	once1.Do(func() {
		conn, err = net.Dial("tcp", addr)
	})
	fmt.Println(conn)
	fmt.Println(err)
}
```

### 标准库中使用Once的场景

标准库内部的`cache`：

```go
// 获取默认的 cache
func Default() *Cache { 
	defaultOnce.Do(initDefaultCache) // 初始化cache
	return defaultCache
}
```

测试的时候初始化的一些测试资源：

```go
// 系统调用时区相关函数
func ForceAusFromTZIForTesting() {
	ResetLocalOnceForTest()
    // 使用Once执行一次初始化
	localOnce.Do(func() { initLocalFromTZI(&aus) })
}
```

除此之外，还有保证只调用一次 copyenv 的 envOnce，strings 包下的 Replacer，time 包中的测试，Go拉取库时的proxy，，net.pipe，crc64，Regexp，…

 在 math/big/sqrt.go 中通过 Once 封装了一个只初始化一次的值：

```go
// 值是3.0或0.0的一个数据结构
var threeOnce struct {
	sync.Once
	v *Float
}
// 返回此数据结构的值，如果还没有初始化为3.0，则初始化
func three() *Float {
	threeOnce.Do(func() { // 使用Once初始化
		threeOnce.v = NewFloat(3.0)
	})
	return threeOnce.v
}
```

**Once 常常用来初始化单例资源，或者并发访问只需初始化一次的共享资源，或者在测试的时候初始化一次测试资源**

### Once的实现

一个正确的 Once 实现要使用一个互斥锁，这样初始化的时候如果有并发的 goroutine，就会进入doSlow 方法。

使用互斥锁加上双检查机制实现Once：

```go
type Once struct {
	done uint32
	m    sync.Mutex
}
func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 0 {
		o.doSlow(f)
	}
}
func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	// 双检查
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
func main() {
	var once Once
	var addr = "baidu.com:80"
	var conn net.Conn
	var err error
	once.Do(func() {
		conn, err = net.Dial("tcp", addr)
	})
	fmt.Println(conn)
	fmt.Println(err)
	once.Do(func() {
		fmt.Println("二次执行")
	})
}
/*
&{{0xc000096000}}
<nil>
*/
```

### Once可能出现的错误

- 死锁
- 未初始化

#### 死锁

```go
// Lock的递归调用导致的死锁
func main() {
	var once sync.Once
	once.Do(func() {
		once.Do(func() {
			fmt.Println("初始化...")
		})
	})
}
// fatal error: all goroutines are asleep - deadlock!
```

#### 未初始化 

调用`f()`时panic或初始化资源失败，once默认执行成功，再次调用`Do()`也不会执行`f()`

```go
func main() {
	var once sync.Once
	var insConn net.Conn
	once.Do(func() {
		fmt.Println("1")
		insConn, _ = net.Dial("tcp", "www.instagram.com:80")
		fmt.Println(insConn)
	})
	once.Do(func() {
		fmt.Println("2")
		insConn, _ = net.Dial("tcp", "www.instagram.com:80")
	})
	insConn.Write([]byte("123"))
	io.Copy(os.Stdout, insConn)
	fmt.Println("end...")
}
/*
运行结果：
1
<nil>
panic: runtime error: invalid memory address or nil pointer dereference
[signal 0xc0000005 code=0x0 addr=0x50 pc=0x9e0666]
*/
```

### 拓展Once

扩展的sync.Once,提供了一个Done()方法：

```go
// Once是 一个扩展的sync.Once,提供了一个Done()方法
type Once struct {
	sync.Once
}
// Done 返回once是否执行过
// 如果执行过返回true
// 未执行或正在执行返回false
func (o *Once) Done() bool {
	return atomic.LoadUint32((*uint32)(unsafe.Pointer(&o.Once))) == 1
}
func main() {
	var once Once
	fmt.Println(once.Done()) // false
	once.Do(func() {
		time.Sleep(1 * time.Second)
	})
	fmt.Println(once.Done()) // true
}
```

一个功能更加强大的Once：

既可以返回当前调用 Do 方法是否正确完成，还可以在初始化失败后调用 Do 方法再次尝试初始化，直到初始化成功才不再初始化了。

```go
// 一个功能更加强大的Once
// 既可以返回当前调用 Do 方法是否正确完成，还可以在初始化失败后调用 Do 方法再次尝试初始化，直到初始化成功才不再初始化了。
type Once struct {
	m    sync.Mutex
	done uint32
}

// 传入的f()有返回值，如果初始化失败需要返回err
// Do会把err返回给调用者
func (o *Once) Do(f func() error) error {
	if atomic.LoadUint32(&o.done) == 1 {
		return nil
	}
	return o.slowDo(f)

}
func (o *Once) slowDo(f func() error) error {
	o.m.Lock()
	defer o.m.Unlock()
	var err error
	// 双检查，还没有初始化
	if o.done == 0 {
		err = f()
		if err == nil {
			// 初始化成功才标记
			atomic.StoreUint32(&o.done, 1)
		}
	}
	return err
}

func main() {
	var once Once
	var insConn net.Conn
	var err error
	err = once.Do(func() error {
		fmt.Println("1")
		insConn, err = net.Dial("tcp", "www.instagram.com:80")
		return err
	})
	err = once.Do(func() error {
		fmt.Println("2")
		insConn, err = net.Dial("tcp", "www.instagram.com:80")
		return err
	})
	if err != nil {
		fmt.Println(err)
		return
	}
	insConn.Write([]byte("123"))
	io.Copy(os.Stdout, insConn)
	fmt.Println("end...")
}

/*
运行输出：
1
2
dial tcp 162.125.2.5:80: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
*/
```

###总结

关于Once：

- 使用场景：Once常常用来初始化单例资源，或者并发访问只需要初始化一次的共享资源，或者在测试的时候初始化一次测试资源。
- 实现 ：一个正确的Once实现要使用一个互斥锁，这样初始化的时候，如果有并发的goroutine就会进入doSlow方法
- 2种错误：
  - 死锁：解决方案：不要在f()中调用这个Once,不管直接的还是间接的。
  - 未初始化：解决方案，可以自己实现一个类似Once的并发原语，既可以返回当前调用Do方法是否正确 完成，还可以在初始化失败后调用Do方法，再次尝试初始化 ，直到初始化成功后才不再初始化了。

最后，**一旦你遇到只需要初始化一次的场景，首先想到的就应该是 Once 并发原语。**
