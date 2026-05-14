---
title: "Golang正则regexp方法示例"
description:  "Golang正则regexp方法示例"
date: 2022-02-18 18:10:00
slug: golang-regex-replace-example
image:
categories:
    - Go
tags: ["Golang"]
---

- regexp.MatchString() 用来匹配子字符串。
- Compile() 或者 MustCompile()创建一个编译好的正则表达式对象。
- FindString 查找字符串
- FindStringIndex 得到匹配字符串的索引
- FindStringSubmatch() 除了返回匹配的字符串外，还会返回子表达式的匹配项。
- FindString方法的All版本，它返回所有匹配的字符串的slice。如果返回nil值代表没有匹配的字符串。
- ReplaceAllString 用来替换所有匹配的字符串，返回一个源字符串的拷贝。

```go
package main

import (
	"fmt"
	"regexp"
)

/*
regexp.MatchString() 用来匹配子字符串。
*/
func MatchString_Demo() {
	str := "Golang regular expressions example"
	math, err := regexp.MatchString(`^Golang`, str)
	fmt.Println(math, err) // true <nil>
}

/*
Compile() 或者 MustCompile()创建一个编译好的正则表达式对象。
假如正则表达式非法，那么Compile()方法回返回error,而MustCompile()编译非法正则表达式时不会返回error，而是回panic。
如果你想要很好的性能，不要在使用的时候才调用Compile()临时进行编译，而是预先调用Compile()编译好正则表达式对象：
*/
func Compile_MustCompile() {
	regexp1, err := regexp.Compile("Gola([a-z]+)g")
	regexp2 := regexp.MustCompile("Gola([a-z]+)g")
	fmt.Println(regexp1, err) // Gola([a-z]+)g <nil>
	fmt.Println(regexp2)      // Gola([a-z]+)g
}

/*
FindString 查找字符串
FindString()用来返回第一个匹配的结果。
如果没有匹配的字符串，那么它回返回一个空的字符串，当然如果你的正则表达式就是要匹配空字符串的话，它也会返回空字符串。
使用 FindStringIndex 或者 FindStringSubmatch可以区分这两种情况。下面是FindString()的例子：
*/
func FindString_Demo() {
	str := "Golang expressions example"
	regexp, _ := regexp.Compile("Gola([a-z]+)g")
	res := regexp.FindString(str)
	fmt.Println(res) // Golang
}

/*
FindStringIndex 得到匹配字符串的索引
FindStringIndex()可以得到匹配的字符串在整体字符串中的索引位置。如果没有匹配的字符串，它回返回nil值。
*/
func FindStringIndex_Demo() {
	str := "Golang regular expressions example"
	regexp, _ := regexp.Compile("exp")
	math := regexp.FindStringIndex(str)
	fmt.Println(math) // [15 18]
}

/*
FindStringSubmatch
FindStringSubmatch() 除了返回匹配的字符串外，还会返回子表达式的匹配项。
如果没有匹配项，则返回nil值。
*/
func FindStringSubmatch_Demo() {
	str := "Golang regular expressions example"
	regexp, _ := regexp.Compile("p([a-z]+)e")
	math := regexp.FindStringSubmatch(str)
	fmt.Println(math) // [pre r]
}

/*
FindString方法的All版本，它返回所有匹配的字符串的slice。如果返回nil值代表没有匹配的字符串。
*/
func FindAllString_Demo() {
	str := "Golang regular expressions example"
	regexp, _ := regexp.Compile("p([a-z]+)e")
	math := regexp.FindAllString(str, 2)
	fmt.Println(math) // [pre ple]
	math = regexp.FindAllString(str, 3)
	fmt.Println(math) // [pre ple]
}

/*
ReplaceAllString 用来替换所有匹配的字符串，返回一个源字符串的拷贝。
*/
func ReplaceAllString_Demo() {
	str := "Golang regular expressions example"
	regexp, _ := regexp.Compile("examp([a-z]+)e")
	math := regexp.ReplaceAllString(str, "tutorial")
	fmt.Println(math) // Golang regular expressions tutorial
}

func main() {
	MatchString_Demo()
	Compile_MustCompile()
	FindString_Demo()
	FindStringIndex_Demo()
	FindStringSubmatch_Demo()
	FindAllString_Demo()
	ReplaceAllString_Demo()
}

```

### 总结

正则表达式提供了16个类似的查找方法，格式为Find(All)?(String)?(Submatch)?(Index)?。

当方法名中有All的时候,它回继续查找非重叠的后续的字符串，返回slice
当方法名中有String的时候,参数类型是字符串，否则是byte slice
当方法名中有Submatch的时候, 还会返回子表达式(capturing group)的匹配项