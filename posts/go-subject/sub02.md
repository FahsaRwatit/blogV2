---
title: 判断字符串中字符是否全都不同
description: 判断字符串中字符是否全都不同
date: 2020-03-16 15:10:00
slug: sub02
image:
categories:
    - Go
tags: ["Go"]

---

## 判断字符串中字符是否全都不同

请实现⼀个算法，确定⼀个字符串的所有字符是否全都不同。这⾥我们要求不允许使⽤额外的存储结构。

给定⼀个string，请返回⼀个bool值，true代表所有字符全都不同，false代表存在相同的字符。

保证字符串中的字符为ASCII字符。

字符串的⻓度⼩于等于3000。

```go
Strings.count()函数
func Count(s, substr string) int
s  表示原字符串。
substr  表示要检索的字符串。
```

**思路**：ASCII字符一共256个，其中128个为常用字符，可以在键盘上输入，128后无法查找

全部不同。也就是字符串中的字符没有重复的。

不准使⽤额外的储存结构。

字符串⼩于等于3000。

### 方法一

```go
package main

import (
	"fmt"
	"strings"
)
/*
判断字符串中字符是否全都不同

请实现⼀个算法，确定⼀个字符串的所有字符是否全都不同。这⾥我们要求不允许使⽤额外的存储结构。
给定⼀个string，请返回⼀个bool值，true代表所有字符全都不同，false代表存在相同的字符。
保证字符串中的字符为ASCII字符。
字符串的⻓度⼩于等于3000。

Strings.count()函数

func Count(s, substr string) int
s	表示原字符串。
substr	表示要检索的字符串。


思路
四点：

ASCII字符一共256个，其中128个为常用字符，可以在键盘上输入，128后无法查找
全部不同。也就是字符串中的字符没有重复的。
不准使⽤额外的储存结构。
字符串⼩于等于3000。


*/

func main() {
	str := "qazzwse"
	fmt.Println(isUniqueString(str))
	fmt.Println(isUniqueString2(str))

}
// strings.count() 用来判断一个字符串包含另外一个字符串的数量
func isUniqueString(str string) bool {
	if strings.Count(str, "") > 3000 {
		return false
	}
	for _, v := range(str) {
		if v > 127 {
			return false
		}
		if strings.Count(str, string(v)) > 1 {
			return false
		}
	}
	return true
}
```

### 方法二

```go
// strings.Index strings.LastIndex  指定字符串在另外一个字符串的索引位置，第一次发现和最后一次发现
func isUniqueString2(str string) bool {
	if strings.Count(str, "") > 3000 {
		return false
	}
	for k, v := range str {
		if v > 127 {
			return false
		}
		if strings.Index(str, string(v)) != k {
			return false
		}
	}
	return true
}
```
