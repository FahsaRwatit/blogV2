---
title: "Golang中使用Redis连接池"
description: "Golang中使用Redis连接池"
date: 2022-5-26 15:10:00
slug: golang-redispool-base
image:
categories:
    - Go
tags: ["Go", "redigo"]


---

使用 `github.com/garyburd/redigo/redis`中的API，获取 Redis 连接池中的实例

### 相关参数

- MaxIdle: 最大空闲连接数

- MaxActive:最大连接数，一般为0，代表不限制

- IdleTimeout：表示空闲连接保活时间，超过该时间后，连接会自动关闭

- Dial：是必须要实现的，就是调用普通的的redis.Dial即可。

- TestOnBorrow：在获取conn的时候会调用一次这个方法，来保证连接可用（其实也不是一定可用，因为test成功以后依然有可能被干掉），这个方法是可选项，一般这个方法是去调用一个redis的ping方法，看项目需求了，如果并发很高，想极限提高速度，这个可以不设置。如果想增加点连接可用性，还是加上比较好。看个人取舍了。

具体实现代码如下：

```go
package main

import (
	"fmt"
	"time"

	"github.com/garyburd/redigo/redis"
)

var (
	pool      *redis.Pool
	redisHost = "ip:端口"
	redisPass = "密码"
)

// 默认用户名为""
// redis-cli -h host -p port -a password
// Dial拨号
// newRedisPool : 创建redis连接池
func newRedisPool() *redis.Pool {
	return &redis.Pool{
		MaxIdle:     50,                // MaxIdle: 最大空闲连接数
		MaxActive:   30,                //  MaxActive:最大连接数，一般为0，代表不限制；
		IdleTimeout: 300 * time.Second, // 表示空闲连接保活时间，超过该时间后，连接会自动关闭
		Dial: func() (redis.Conn, error) {
			// 1、打开连接
			c, err := redis.Dial("tcp", redisHost)
			if err != nil {
				fmt.Println(err)
				return nil, err
			}

			// 访问认证
			if _, err = c.Do("AUTH", redisPass); err != nil {
				c.Close()
				return nil, err
			}
			return c, nil
		},
		TestOnBorrow: func(conn redis.Conn, t time.Time) error {
			if time.Since(t) < time.Minute {
				return nil
			}
			_, err := conn.Do("PING")
			return err
		},
	}
}

func init() {
	fmt.Println("初始化...")
	pool = newRedisPool()
}

func RedisPool() *redis.Pool {
	return pool
}

func main() {
	// // 获得redis的一个连接
	rdb := RedisPool().Get()
	defer rdb.Close()

	res, err := rdb.Do("Set", "test1", 100)
	if err != nil {
		fmt.Printf("set test1 failed, err:%v\n", err)
		return
	}
	fmt.Println(res)
	val, err := redis.Int(rdb.Do("Get", "test1"))
	if err != nil {
		fmt.Printf("get test1 failed, err:%v\n", err)
		return
	}
	fmt.Println("test1", val)

}

```

