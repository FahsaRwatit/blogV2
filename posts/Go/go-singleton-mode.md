---
title: "Go实现单例模式"
description: "Go实现单例模式"
date: 2021-04-10 18:10:00
slug: go-singleton-mode
image:
categories:
    - Go
tags: ["Go", "设计模式"]


---

单例模式（Singleton Pattern），是最简单的一个模式，比较适合全局共享一个实例，且只需要被初始化一次的场景，例如数据库实例、全局配置、全局任务池等。

单例模式又分为饿汉方式和懒汉方式。

### 饿汉方式

饿汉方式指全局的单例实例在包被加载时创建。

```go
type singleton struct {

}
var ins *singleton = &singleton{}

func getIns() *singleton {
	fmt.Println("getIns")
	return ins
}
```

### 懒汉方式

懒汉方式指全局的单例实例在第一次被使用时创建

懒汉方式是开源项目中使用最多的，但它的缺点是非并发安全，在实际使用时需要加锁。

```go
// 为了解决懒汉方式非并发安全的问题，需要对实例进行加锁，
import (
	"fmt"
	"sync"
)

type singleton struct {

}
var ins *singleton
var mu sync.Mutex

func getIns() *singleton{
	if ins == nil {
		mu.Lock()
		if ins == nil {
			fmt.Println("创建")
			ins = &singleton{}
		}
		mu.Unlock()
	}
	return ins
}
```



**一种更加优雅的方式，建议采用这种**

once.Do可以确保 ins 实例全局只被创建一次，once.Do 函数还可以确保当同时有多个创建动作时，只有一个创建动作在被执行。

```go
type singleton struct {

}
var ins *singleton
var once sync.Once
func getIns() *singleton {
	once.Do(func() {
		fmt.Println("创建")
		ins = &singleton{}
	})
	return ins
}

func main() {
	for i := 1; i < 100; i++ {
		getIns()
	}
}
```

