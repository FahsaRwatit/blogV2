---
title: "Golang中的方法集"
description: "Golang中的方法集"
date: 2022-08-11 18:10:00
slug: golang-method-set
image:
categories:
    - Go
tags: ["Go"]

---

**go语言的方法定义：**


1、参数 receiver 可任意命名，如方法中，不使用参数，可以省略参数名。

2、参数 receiver 类型可以是 T 或 *T，但类型T不能为接口或指针类型。

3、不支持方法重载。

4、可以实例value或pointer调用全部的方法，编译器会自动转换。

**实体类型实现接口和以指针类型实现接口的区别**

1、若以实体类型（T）实现接口，不管是T类型的值，还是T类型的指针，都实现了该接口。

2、若以指针类型（*T）实现接口，只有T类型的指针，才实现了该接口。

**go语言通过嵌入组合，来实现继承的行为。**

**值类型(T)嵌入和指针类型(*T)嵌入的区别：**

type Student1 struct {
   Person //值类型的嵌入
}
type Student2 struct {
   *Person //指针类型的嵌入
}

要理解这个区别，就有知道go语言中类型的默认值。如下：

1、数值类型（如int8、int16、uint等），默认值为0；

2、布尔类型，默认值为false；

3、字符串类型，默认值为""

4、指针、通道、切片、字典等，默认值为nil

5、复合类型的默认值，为所包含类型的默认值。


针对指针嵌入类型，在使用前，需要赋值。

**嵌入和方法集的关系：**

1、类型 S 包含匿名字段 T，则 S和*S 方法集包含 T 方法。

2、类型 S 包含匿名字段 *T，则 S和 *S 方法集包含 T + *T 方法。

3、不管嵌入的是T还是*T，*S方法集，包含 T + *T 方法。

• 类型 T 方法集包含全部 receiver T 方法。
• 类型 *T 方法集包含全部 receiver T + *T 方法。
• 如类型 S 包含匿名字段 T，则 S 和 *S 方法集包含 T 方法。 
• 如类型 S 包含匿名字段 *T，则 S 和 *S 方法集包含 T + *T 方法。 
• 不管嵌入 T 或 *T，*S 方法集总是包含 T + *T 方法。

**方法集：**

方法集合也是用来判断一个类型是否实现了某接口类型的唯一手段，可以说，“方法集合决定了接口实现”。

所谓的方法集合决定接口实现的含义就是：如果某类型 T 的方法集合与某接口类型的方法集合相同，或者类型 T 的方法集合是接口类型 I 方法集合的超集，那么我们就说这个类型 T 实现了接口 I。
或者说，方法集合这个概念在 Go 语言中的主要用途，就是用来判断某个类型是否实现了某个接口。

###示例

dumpMethodSet，用于输出一个非接口类型的方法集合

```go
package main

import (
	"fmt"
	"reflect"
)

func dumpMethodSet(i interface{}) {
	dynTyp := reflect.TypeOf(i)

	if dynTyp == nil {
		fmt.Printf("there is no dynamic type\n")
		return
	}

	n := dynTyp.NumMethod()
	if n == 0 {
		fmt.Printf("%s's method set is empty!\n", dynTyp)
		return
	}

	fmt.Printf("%s's method set:\n", dynTyp)
	for j := 0; j < n; j++ {
		fmt.Println("-", dynTyp.Method(j).Name)
	}
	fmt.Printf("\n")
}

type T struct{}

func (T) M1() {}
func (T) M2() {}

func (*T) M3() {}
func (*T) M4() {}

type S T

/*
Go 语言规定，*T 类型的方法集合包含所有以 *T 为 receiver 参数类型的方法，以及所有以 T 为 receiver 参数类型的方法。
*/
func main() {
	var n int
	dumpMethodSet(n)
	dumpMethodSet(&n)

	var t T
	dumpMethodSet(t)
	dumpMethodSet(&t)
	/*
		int's method set is empty!
		*int's method set is empty!
		main.T's method set:
		- M1
		- M2

		*main.T's method set:
		- M1
		- M2
		- M3
		- M4

	*/

	// 这个 S 类型包含哪些方法呢？*S 类型又包含哪些方法呢？
	/*
		S 类型 和 *S 类型都没有包含方法，因为type S T 定义了一个新类型。
		但是如果用 type S = T 则S和*S类型都包含两个方法。
	*/
	var s S
	dumpMethodSet(s)  // main.S's method set is empty!
	dumpMethodSet(&s) // *main.S's method set is empty!

	type S2 = T
	var s2 S2
	dumpMethodSet(s2)
	/*
		main.T's method set:
		- M1
		- M2
	*/
	dumpMethodSet(&s2)
	/*
			*main.T's method set:
		- M1
		- M2
		- M3
		- M4
	*/
}

```

**References**

http://www.findme.wang/blog/detail/id/559.html
