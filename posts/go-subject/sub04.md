---
title: 判断两个给定的字符串排序后是否⼀致
description: 判断两个给定的字符串排序后是否⼀致
date: 2020-03-18 15:10:00
slug: sub04
image:
categories:
    - Go
tags: ["Go"]

---

## 判断两个给定的字符串排序后是否⼀致

给定两个字符串，请编写程序，确定其中⼀个字符串的字符重新排列后，能否变成另⼀个字符串。

这⾥规定大小写为不同字符。

给定⼀个string s1和⼀个string s2，请返回⼀个bool，代表两串是否重新排列后可相同。

保证两串的⻓度都⼩于等于5000。

**思路**：

- 2个字符串长度⼩于等于5000

- s1 s2 长度需一致

- 判断s1中每个字符的个数与s2中的相同即可。

### 方法一

```go
package main

import (
	"fmt"
	"strings"
)
/*
判断两个给定的字符串排序后是否⼀致。
给定两个字符串，请编写程序，确定其中⼀个字符串的字符重新排列后，能否变成另⼀个字符串。
这⾥规定大小写为不同字符。
给定⼀个string s1和⼀个string s2，请返回⼀个bool，代表两串是否重新排列后可相同。
保证两串的⻓度都⼩于等于5000。

思路：
2个字符串长度⼩于等于5000
s1 s2 长度需一致
判断s1中每个字符的个数与s2中的相同即可。

*/
func main() {
	s1 := "qazwsxwsaz"
	s2 := "qazwsxwsza"

	fmt.Println(isRegroup(s1, s2))
	
}
func isRegroup(s1 string, s2 string) bool {
	if len(s1) > 5000 || len(s2) > 5000 || len(s1) != len(s2) {
		return false
	}
	for _, v := range(s1) {
		if strings.Count(s1, string(v)) != strings.Count(s2, string(v)) {
			return false
		}
	}
	return true
}
```

