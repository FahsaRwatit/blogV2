---
title: "sonyflake基于雪花算法生成分布式ID"
description: "sonyflake基于雪花算法生成分布式ID"
date: 2022-03-12 15:10:00
slug: "snowflake-sonyflake"
image: ""
categories:
    - Go
tags: ["Go"]

---

SonyFlake是索尼对Twitter开源的分布式ID生成算法-雪花算法（SnowFlake）改进后的算法。

基本实现思路和 SnowFlake 差不多，但也有差异，如下：

- 如下为 snowflake 的ID 组成

|              |                  |                |                    |
| ------------ | ---------------- | -------------- | ------------------ |
| 1 Bit Unused | 41 Bit Timestamp | 10 Bit Node ID | 12 Bit Sequence ID |

- 如下为 Sonyflake 的ID 组成

|              |                  |                   |                   |
| ------------ | ---------------- | ----------------- | ----------------- |
| 1 Bit Unused | 39 Bit Timestamp | 8 Bit Sequence ID | 16 Bit Machine ID |

**解析**

- Unused: 未使用, 二进制中最高位为1的都是负数, 我们生成的id一般都是整数, 所以这个最高位固定是0。
- Timestamp: 时间戳.
- Sequence ID: 由程序在运行期生成。
- Node ID: 机器编号的位长。
- Machine ID: 其实就是 Node ID。

#### 安装

go get -u github.com/sony/sonyflake

#### 导入

import "github.com/sony/sonyflake"

**注：**如果作为Mysql主键使用，字段类型需使用 bigint 类型

### 示例

```go
package snowflake

import (
	"fmt"
	"time"

	"github.com/sony/sonyflake"
)

var (
	sonyFlake *sonyflake.Sonyflake
	// 定义一个全局的 machineID 模拟获取
	// 现实环境中应从 zk 或 etcd 中获取
	sonyMachineID uint16
)

// 获取机器编码ID
func getMachineID() (uint16, error) {
	return sonyMachineID, nil
}

// 需传入当前的机器ID
func Init(machineId uint16) (err error) {
	sonyMachineID = machineId
	// 格式化时间 1月2号下午3时4分5秒  2006年， 开始时间 2020-01-01
	t, _ := time.Parse("2006-01-02", "2020-01-01")
	settings := sonyflake.Settings{
		StartTime: t,
		MachineID: getMachineID,
	}
	sonyFlake = sonyflake.NewSonyflake(settings)
	return
}

func GetID() (id uint64, err error) {
	if sonyFlake == nil {
		err = fmt.Errorf("sony flake not inited")
		return
	}
	id, err = sonyFlake.NextID()
	return
}

```



```go
// 雪花算法
if err := snowflake.Init(1); err != nil {
    fmt.Printf("init snowflake failed, err:%v\n", err)
    return
}


// 雪花算法 生成userid
userID, err := snowflake.GetID()
if err != nil {
    return ErrorGenIDFailed
}

```

