---
title: 翻转字符串
description: 翻转字符串
date: 2020-03-17 15:10:00
slug: sub03
image:
categories:
    - Go
tags: ["Go"]

---

## 交替打印数字和字母

请实现⼀个算法，在不使⽤【额外数据结构和储存空间】的情况下，翻转⼀个给定的字
符串(可以使⽤单个过程变量)。
给定⼀个string，请返回⼀个string，为翻转后的字符串。保证字符串的⻓度⼩于等于
5000。

**思路**：
翻转字符串其实是将⼀个字符串以中间字符为轴，前后翻转，即将str[len]赋值给str[0],
将str[0] 赋值 str[len]。

### 方法一

```go
package main

import (
	"fmt"
)
/*
请实现⼀个算法，在不使⽤【额外数据结构和储存空间】的情况下，翻转⼀个给定的字
符串(可以使⽤单个过程变量)。
给定⼀个string，请返回⼀个string，为翻转后的字符串。保证字符串的⻓度⼩于等于
5000。

思路：
翻转字符串其实是将⼀个字符串以中间字符为轴，前后翻转，即将str[len]赋值给str[0],
将str[0] 赋值 str[len]。

*/
func main() {
	str := "wsxqaz"
	fmt.Println(reverString(str)) // zaqxsw true

}

func reverString(s string) (string, bool) {
	str := []rune(s)
	if len(str) > 5000 {
		return s, false
	}
	for i := 0; i < len(str) / 2; i++ {
		str[i], str[len(str) - 1 - i] = str[len(str) - 1 - i], str[i]
	}
	return string(str), true

}
```
