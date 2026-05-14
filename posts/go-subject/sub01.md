---
title: 交替打印数字和字母
description: 交替打印数字和字母
date: 2020-03-15 15:10:00
slug: sub01
image:
categories:
    - Go
tags: ["Go"]

---

## 交替打印数字和字母

使用两个 goroutine 交替打印序列，一个 goroutinue 打印数字， 另外一个goroutine打印字母， 最终效果如下:

  ```go
  12AB34CD56EF78GH910IJ1112KL1314MN1516OP1718QR1920ST2122UV2324WX2526YZ2728
  ```

**思路**：使用 channel 来控制打印的进度。使用两个 channel ，来分别控制数字和字母的打印序列，

   数字打印完成后通过 channel 通知字母打印, 字母打印完成后通知数字打印，然后周而复始的工作。

### 方法一

```go
package main

import (
	"fmt"
	// "sync"
)

func main() {
	letter, number := make(chan bool), make(chan bool)
	done := make(chan int)
	// fmt.Println(letter)
	// fmt.Println(number)

	go func() {
		for i := 1; i <= 28; i += 2 {
			<-number
			fmt.Print(i)
			fmt.Print(i+1)

			if i >= 27 {
				// 结束
				done <- 1
			}
			letter <- true
		}
	}()

	go func() {
		for i := 'A'; i <= 'Z'; i += 2 {
			<-letter
			fmt.Print(string(i))
			fmt.Print(string(i+1))
			number <- true
		}
	}()
	number <- true
	<-done
}
```

### 方法二

```go
func main() {
	/*
	使用两个 goroutine 交替打印序列，一个 goroutinue 打印数字， 另外一个goroutine打印字母， 最终效果如下:
	12AB34CD56EF78GH910IJ1112KL1314MN1516OP1718QR1920ST2122UV2324WX2526YZ2728

	思路：使用 channel 来控制打印的进度。使用两个 channel ，来分别控制数字和字母的打印序列，
	 数字打印完成后通过 channel 通知字母打印, 字母打印完成后通知数字打印，然后周而复始的工作。

	*/
	letter, number := make(chan bool), make(chan bool)
	fmt.Println(letter)
	fmt.Println(number)

	var wg sync.WaitGroup

	wg.Add(1)
	// 数字
	go func(){
		i := 1
		for {
			select {
			case <-number:
				fmt.Print(i)
				i++
				fmt.Print(i)
				i++
				letter <- true
			}
			
		}
	}()
	// 放在 数字 goroutine 前会产生阻塞
	number <- true

	// 字母
	go func(wg *sync.WaitGroup){
		i := 'A'
		for {
			select {
			case <-letter:
				if i >= 'Z' {
					wg.Done()
					fmt.Println("")
					return
				}
				fmt.Print(string(i))
				i++
				fmt.Print(string(i))
				i++
				number <- true
			}
		}
	}(&wg)

	wg.Wait()

	fmt.Println("end...")

}
/*
12AB34CD56EF78GH910IJ1112KL1314MN1516OP1718QR1920ST2122UV2324WX2526YZ2728
*/
```
