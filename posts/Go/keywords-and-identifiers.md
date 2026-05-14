---
title: "Golang中关键词和预定义标识符"
description: "Golang中关键词和预定义标识符"
date: 2022-06-08 18:10:00
slug: keywords-and-identifiers
image:
categories:
    - Go
tags: ["Go"]

---

### 25个关键字

| -        | -           | -      | -         | -      |
| -------- | ----------- | ------ | --------- | ------ |
| break    | default     | func   | interface | select |
| case     | defer       | go     | map       | struct |
| chan     | else        | goto   | package   | switch |
| const    | fallthrough | if     | range     | type   |
| continue | for         | import | return    | var    |

### 37个预定义标识符

| 类型              | 名称                                                         |
| ----------------- | ------------------------------------------------------------ |
| 常量(4个)         | true、false、iota、nil                                       |
| 整型(11个)        | int、int8、in16、int32、int64、uint、uint8、uint16、uint32、uint64、uintptr |
| 浮点型(2个)       | float32、float64                                             |
| 复数型（2个）     | complex64、complex128                                        |
| 布尔型（1个）     | bool                                                         |
| 字节（1个）       | byte                                                         |
| 特殊整型（1个）   | rune（为int32，用来区别字符型和整数型）                      |
| 字符串类型（1个） | string                                                       |
| 错误类型（1个）   | error                                                        |
| 函数（13个）      | make、len、cap、new、append、copy、close、delete、complex、real（实部）、imag（虚部）、panic、recover |