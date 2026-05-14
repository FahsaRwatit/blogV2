---
title: "GoLang的面向对象特性"
description: "GoLang 的面向对象特性"
date: 2022-08-11 18:10:00
slug: golang-object
image:
categories:
    - Go
tags: ["Go"]
---

```go
package main

import "fmt"

// Go 的面向对象特性

// 接口
/*
接口使用 interface 关键字声明，任何实现接口定义方法的类都可以实例化该接口。
接口和实现类之间没有任何依赖，你可以实现一个新的类当做 Sayer 来使用，而不需要依赖 Sayer 接口，
也可以为已有的类创建一个新的接口，而不需要修改任何已有的代码，和其他静态语言相比，这可以算是 golang 的特色了吧
*/
type Sayer interface {
	Say(message string)
	SayHi()
}

// 继承，使用组合的方式实现继承
// Dog 将继承 Animal 的 Say 方法，以及其成员 Name
type Animal struct {
	Name string
}

func (a *Animal) Say(message string) {
	fmt.Printf("Animal[%v] say: %v\n", a.Name, message)
}

func (a *Animal) SayHi() {
	a.Say("Hi")
}

type Dog struct {
	Animal
}

// 覆盖，子类可以重新实现父类的方法
// override Animal.Say
// Dog.Say 将覆盖 Animal.Say
func (d *Dog) Say(message string) {
	fmt.Printf("Dog[%v] say: %v\n", d.Name, message)
}

// 同样子类继承的父类的方法 引用的父类的其他方法也没有多态特性

func main() {
	// 多态
	// 接口可以用任何实现该接口的指针来实例化
	var sayer Sayer
	sayer = &Dog{Animal{Name: "Mai"}}
	sayer.Say("Hello Golang") // Dog[Mai] say: Hello Golang

	// 但是不支持父类指针指向子类，下面这种写法是不允许的
	// var animal *Animal
	// animal = &Dog{Animal{Name: "Mai"}} //  cannot use &Dog{…} (value of type *Dog) as type *Animal in assignment

	// 同样子类继承的父类的方法引用的父类的其他方法也没有多态特性
	var sayer1 Sayer
	sayer1 = &Dog{Animal{Name: "Mario"}}
	sayer1.Say("hello world") // Dog[Mario] say: hello world
	sayer1.SayHi()            // Animal[Mario] say: Hi

	/*
		上面这段代码中，子类 Dog 没有实现 SayHi 方法，调用的是从父类 Animal.SayHi，
		而 Animal.SayHi 调用的是 Animal.Say 而不是Dog.Say，这一点和其他面向对象语言有所区别，
		需要特别注意，但是可以用下面的方式来实现类似的功能，以提高代码的复用性
	*/
	var sayer2 Sayer

	sayer2 = &Cat{Animal{Name: "Ann"}}
	sayer2.Say("hello world") // Cat[Ann] say: hello world
	sayer2.SayHi()            // Cat[Ann] say: Hi

}

func SayHi(s Sayer) {
	s.Say("Hi")
}

type Cat struct {
	Animal
}

func (c *Cat) Say(message string) {
	fmt.Printf("Cat[%v] say: %v\n", c.Name, message)
}

func (c *Cat) SayHi() {
	SayHi(c)
}
```