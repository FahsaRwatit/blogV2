---
title: "[LeetCode]找出数组中重复的数字"
description:  "[LeetCode]找出数组中重复的数字"
date: 2021-06-08 18:10:00
slug: findRepeatNumber
image:
categories:
    - Go
tags: ["Golang", "LeetCode"]

---

- 1、先排序后，判断前后是否相等
- 2、使用map
- 3、原地置换

```go
package main

import "fmt"

/*
找出数组中重复的数字。

在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。
数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。
请找出数组中任意一个重复的数字。

示例 1：

输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3

链接：https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof
*/
// 1、先排序后，判断前后是否相等
func findRepeatNumber1(nums []int) int {
	fmt.Println(nums)
	for i := 0; i < len(nums); i++ {
		for j := 0; j < i; j++ {
			if nums[i] < nums[j] {
				nums[i], nums[j] = nums[j], nums[i]
			}
		}
	}
	for k := range nums {
		if k == 0 {
			continue
		}
		if nums[k-1] == nums[k] {
			return nums[k-1]
		}
	}
	return -1
}

// 使用map
func findRepeatNumber2(nums []int) int {
	data := make(map[int]bool, len(nums))
	for k := range nums {
		if data[nums[k]] {
			return nums[k]
		}
		data[nums[k]] = true
	}
	return -1
}

// 原地置换
/*
利用的条件为：nums 里的所有数字都在 0～n-1 的范围内
遍历数组，根据当前位置k的值是否为k,如果不是则换位置，交换位置时判断要交换的值是否相等。
*/
func findRepeatNumber(nums []int) int {
	for k := range nums {
		for k != nums[k] {
			if nums[k] == nums[nums[k]] {
				return nums[k]
			}
			nums[k], nums[nums[k]] = nums[nums[k]], nums[k]
		}
	}
	return -1
}

func main() {
	var nums = []int{2, 3, 1, 0, 3, 5, 2}
	var num = findRepeatNumber(nums)
	fmt.Println(num)
}

```